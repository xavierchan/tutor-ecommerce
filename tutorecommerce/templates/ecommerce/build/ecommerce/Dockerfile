FROM docker.io/ubuntu:18.04

RUN apt update && \
  apt install -y git-core language-pack-en libmysqlclient-dev libssl-dev python python-pip python-virtualenv

# Create python venv
RUN mkdir /openedx/ && virtualenv /openedx/venv/
ENV PATH "/openedx/venv/bin:$PATH"
RUN pip install --upgrade pip setuptools

# Install a recent version of nodejs
RUN pip install nodeenv
RUN nodeenv /openedx/nodeenv --node=8.9.3 --prebuilt
ENV PATH /openedx/nodeenv/bin:${PATH}

# Install ecommerce
ARG ECOMMERCE_REPOSITORY=https://github.com/edx/ecommerce.git
ARG ECOMMERCE_VERSION=open-release/ironwood.2
RUN mkdir -p /openedx/ecommerce && \
    git clone $ECOMMERCE_REPOSITORY --branch $ECOMMERCE_VERSION --depth 1 /openedx/ecommerce
WORKDIR /openedx/ecommerce

# nodejs requirements
RUN npm install
RUN ./node_modules/.bin/bower install --allow-root
# Setup minimal yml config file, which is required by production settings
RUN echo "{}" > /openedx/config.yml
ENV ECOMMERCE_CFG /openedx/config.yml
# python requirements
RUN pip install -r requirements.txt &&  pip install -r requirements/dev.txt

{% for extra_requirement in ECOMMERCE_EXTRA_PIP_REQUIREMENTS %}RUN pip install {{ extra_requirement }}
{% endfor %}
# Collect static assets
COPY assets.py ./ecommerce/settings/assets.py
ENV DJANGO_SETTINGS_MODULE ecommerce.settings.assets
RUN ./node_modules/.bin/r.js -o build.js
RUN make theme_static
RUN python manage.py collectstatic --noinput
RUN python manage.py compress --force



EXPOSE 8000
CMD gunicorn --bind=0.0.0.0:8000 --workers 2 --max-requests=1000 ecommerce.wsgi:application
