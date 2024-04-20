# Pull base Docker image
FROM python:3.11.2-slim-bullseye AS builder

# Patch with most recent security updates and bug fixes
RUN apt-get update && apt-get upgrade --yes

# Create and switch to regular user
RUN useradd --create-home alanpham
USER alanpham
WORKDIR /home/alanpham

# Isolate Docker image
ENV VIRTUALENV=/home/alanpham/venv
RUN python3 -m venv $VIRTUALENV
ENV PATH="$VIRTUALENV/bin:$PATH"

# Cache project's dependencies
COPY --chown=alanpham pyproject.toml constraints.txt ./

# Update pip and install dependencies
RUN python -m pip install --upgrade pip setuptools && \
    python -m pip install --no-cache-dir -c constraints.txt ".[dev]"

# Run tests as part of the build process
# Copy source code
COPY --chown=alanpham src/ src/
COPY --chown=alanpham test/ test/

# Run tests along with linters and other statuc analysis tools:
RUN python -m pip install . -c constraints.txt && \
    python -m pytest test/unit/ && \
    python -m flake8 src/ && \
    python -m isort src/ --check && \
    python -m black src/ --check --quiet && \
    python -m pylint src/ --disable=C0114,C0116,R1705 && \
    python -m bandit -r src/ --quiet && \
    python -m pip wheel --wheel-dir dist/ . -c constraints.txt

FROM python:3.11.2-slim-bullseye

RUN apt-get update && apt-get upgrade --yes

RUN useradd --create-home alanpham
USER alanpham
WORKDIR /home/alanpham

ENV VIRTUALENV=/home/alanpham/venv
RUN python3 -m venv $VIRTUALENV
ENV PATH="$VIRTUALENV/bin:$PATH"

COPY --from=builder /home/alanpham/dist/page_tracker*.whl /home/alanpham

RUN python -m pip install --upgrade pip setuptools && \
    python -m pip install --no-cache-dir page_tracker*.whl

CMD ["flask", "--app", "page_tracker.app", "run", "--host", "0.0.0.0", "--port", "5000"]
