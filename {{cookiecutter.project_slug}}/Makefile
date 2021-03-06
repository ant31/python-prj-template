.PHONY: clean-pyc clean-build docs clean test test-all
define BROWSER_PYSCRIPT
import os, webbrowser, sys
try:
	from urllib import pathname2url
except:
	from urllib.request import pathname2url

webbrowser.open("file://" + pathname2url(os.path.abspath(sys.argv[1])))
endef
export BROWSER_PYSCRIPT
BROWSER := python -c "$$BROWSER_PYSCRIPT"
VERSION := `cat VERSION`
project := "{{cookiecutter.project_slug}}"

help:
	@echo "clean - remove all build, test, coverage and Python artifacts"
	@echo "clean-build - remove build artifacts"
	@echo "clean-pyc - remove Python file artifacts"
	@echo "clean-test - remove test and coverage artifacts"
	@echo "lint - check style with flake8"
	@echo "test - run tests quickly with the default Python"
	@echo "test-all - run tests on every Python version with tox"
	@echo "coverage - check code coverage quickly with the default Python"
	@echo "docs - generate Sphinx HTML documentation, including API docs"
	@echo "release - package and upload a release"
	@echo "dist - package"
	@echo "install - install the package to the active Python's site-packages"

clean: clean-build clean-pyc clean-test

clean-build:
	rm -fr build/
	rm -fr dist/
	rm -fr .eggs/
	find . -name '*.egg-info' -exec rm -fr {} +
	find . -name '*.egg' -exec rm -f {} +

clean-pyc:
	find . -name '*.pyc' -exec rm -f {} +
	find . -name '*.pyo' -exec rm -f {} +
	find . -name '*~' -exec rm -f {} +
	find . -name '__pycache__' -exec rm -fr {} +

clean-test:
	rm -fr .tox/
	rm -f .coverage
	rm -fr htmlcov/

lint:
	flake8 $(project) tests

test:
	py.test --cov=$(project) --cov-report=html --cov-report=term-missing  --verbose tests

test-all:
	py.test --cov=$(project) --cov-report=html --cov-report=term-missing  --verbose tests

tox:
	tox

coverage:
	coverage run --source $(project) setup.py test
	coverage report -m
	coverage html
	$(BROWSER) htmlcov/index.html

docs: install
	rm -f test1
	sphinx-apidoc  -f -P -o docs/test1 $(project)
	$(MAKE) -C docs clean
	$(MAKE) -C docs html
	$(BROWSER) docs/_build/html/index.html

servedocs: docs
	watchmedo shell-command -p '*.rst' -c '$(MAKE) -C docs html' -R -D .

release: clean
	python setup.py sdist upload
	python setup.py bdist_wheel upload


dist: clean
	python setup.py sdist
	python setup.py bdist_wheel
	ls -l dist

install: clean
	pip install -r requirements.txt
	python setup.py install

flake8:
	python setup.py flake8

coveralls: test
	coveralls

pylint:
	pylint --rcfile=.pylintrc $(project)  -E -r y

pylint-all:
	pylint --rcfile=".pylintrc" $(project)

coveralls: test
	coveralls

dockerfile: clean
	docker build -t {{cookiecutter.docker_registry}}:v$(VERSION) .

dockerfile-canary: clean
	docker build -t {{cookiecutter.docker_registry}}:canary .
	docker push {{cookiecutter.docker_registry}}

dockerfile-push: dockerfile
	docker push {{cookiecutter.docker_registry}}:v$(VERSION)

mypy:
	mypy $(project) --ignore-missing-imports

fmt-ci:
	find . -iname "*.jsonnet" | xargs jsonnet fmt -i -n 2
	find . -iname "*.libsonnet" | xargs jsonnet fmt -i -n 2

gen-ci: fmt-ci
	ffctl gen

check: pylint flake8 mypy yapf-test gen-ci gen-config

prepare: yapf gen-ci check

yapf:
	yapf -r $(project) -i

yapf-diff:
	yapf -r $(project) -d

yapf-test: yapf-diff
	if [ `yapf -r appr -d | wc -l` -gt 0 ] ; then false ; else true ;fi
