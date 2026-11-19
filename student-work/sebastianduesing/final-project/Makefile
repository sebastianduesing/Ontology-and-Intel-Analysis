# DBO Makefile
# Sebastian Duesing
#
# This Makefile is used to build artifacts for the Database Ontology (DBO),
# a provisional ontology demonstrating proposed additions to the Information
# Artifact Ontology (IAO). This file is based on the OBI Makefile.
#
# WARN: This file contains significant whitespace, i.e. tabs!
# Ensure that your text editor shows you those characters.

### Configuration
#
# These are standard options to make Make sane:
# <http://clarkgrubb.com/makefile-style-guide#toc2>

MAKEFLAGS += --warn-undefined-variables
SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c
.DEFAULT_GOAL := all
.DELETE_ON_ERROR:
.SUFFIXES:
.SECONDARY:

### Definitions

SHELL   := /bin/bash
OBO     := http://purl.obolibrary.org/obo
DBO     := https://example.org/dbo
DEV     := $(DBO)/dev
TODAY   := $(shell date +%Y-%m-%d)
TS      := $(shell date +'%d:%m:%Y %H:%M')

### Directories
# 
# This is a temporary place to put things.
build build/views:
	mkdir -p $@


### ROBOT
build/robot.jar: | build
	curl -L -o $@ https://github.com/ontodev/robot/releases/download/v1.9.8/robot.jar

build/robot: build/robot.jar
	curl -L -o $@ https://raw.githubusercontent.com/ontodev/robot/master/bin/robot
	chmod +x $@

ROBOT := java -jar build/robot.jar --prefix 'DBO: https://example.org/dbo/DBO_'

### Imports
# 
# Download files from which terms will be extracted
build/%_source.owl: | build
	curl -L -o $@ http://purl.obolibrary.org/obo/$*.owl

# Use ROBOT to extract terms
build/%_imports.owl: src/ontology/imports/%_input.txt build/%_source.owl | build
	$(ROBOT) extract \
	--input $(word 2,$^) \
	--method MIREOT \
	--upper-term BFO:0000015 \
	--lower-terms $< \
	annotate \
	--ontology-iri "$(DEV)/$(notdir $@)" \
	--version-iri "$(DEV)/$(TODAY)/$(notdir $@)" \
	--output $@

# Ensure serialization is consistent
src/ontology/imports/%_imports.owl: build/%_imports.owl
	$(ROBOT) convert -i build/$*_imports.owl -o $@
	rm build/$*_imports.owl

IMPORT_FILES := $(wildcard src/ontology/imports/*_imports/owl)

.PHONY: imports
	imports: $(IMPORT_FILES)

build/dbo_merged.owl: src/ontology/dbo-external.owl build/*_imports.owl
	$(ROBOT) merge \
	--input $< \
	annotate \
	--ontology-iri "$(DBO)/dbo_merged.owl" \
	--version-iri "$(DBO)/$(TODAY)/dbo_merged.owl" \
	--annotation owl:versionInfo "$(TODAY)" \
	--output $@

### Template
#
# `src/ontology/dbo-edit.tsv` is a ROBOT template containing all DBO terms.
dbo.owl: src/ontology/dbo-edit.tsv build/dbo_merged.owl | build/robot.jar
	echo '' > $@
	$(ROBOT) template --merge-after \
	--input $(word 2,$^) \
	--collapse-import-closure true \
	--template $< \
	annotate \
	--ontology-iri "$(DBO)/dbo.owl" \
	--version-iri "$(DBO)/$(TODAY)/dbo.owl" \
	--annotation owl:versionInfo "$(TODAY)" \
	--output $@
