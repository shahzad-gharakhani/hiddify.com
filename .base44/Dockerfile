# Toolchain for building the Hiddify docs site with `mkdocs serve`.
# Only dependencies live in the image; the app source is bind-mounted at runtime
# so edits are picked up by the dev server without a rebuild.
FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    VIRTUAL_ENV=/opt/venv \
    PATH="/opt/venv/bin:$PATH"

RUN apt-get update && apt-get install -y --no-install-recommends \
      git \
      build-essential \
      curl \
      libcairo2-dev \
      libfreetype6-dev \
      libffi-dev \
      libjpeg-dev \
      libpng-dev \
      libz-dev \
      pngquant \
    && rm -rf /var/lib/apt/lists/*

RUN python -m venv "$VIRTUAL_ENV"

# Mirrors the repo's `make prepare` target. The hiddify forks of mkdocs,
# mkdocs-static-i18n and lunr.py are required: the site uses `materialx.emoji`
# and the `languages="fa"` search index that only exist in those forks.
RUN pip install "mkdocs-material[imaging]==9.2.5" \
        mkdocs-rss-plugin \
        mkdocs-git-revision-date-localized-plugin \
        mkdocs-git-authors-plugin \
        mkdocs-awesome-pages-plugin \
        mkdocs-nav-weight \
        nltk \
        mkdocs-include-markdown-plugin \
        mkdocs-exclude \
 && pip install -U --force-reinstall \
        "git+https://github.com/hiddify/mkdocs" \
        "git+https://github.com/hiddify/mkdocs-static-i18n" \
        "git+https://github.com/hiddify/lunr.py"

WORKDIR /app
EXPOSE 8000
