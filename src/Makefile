#TOP_DIR = $(realpath ..)
TOP_DIR = ..

# Specify the document version.
# This can be overridden when invoked from the automated build process.
DOC_VERSION = X.Y.Z

# Specify whether a spelling error is an error (as opposed to a warning).
# This can be overridden when invoked from the automated build process.
# Please resist the temptation to set this variable to zero. ;-)
# DOC_SPELLCHECK_MUST_PASS = 0
DOC_SPELLCHECK_MUST_PASS = 1

# This can be overridden when invoked from the automated build process.
INSTALL_DIR = $(TOP_DIR)/install

# All of the Markdown source files (that are not generated during build).
SOURCES = \
  main.md \

################################################################################
# Define primary targets.
################################################################################

# The all target builds the document in a minimal set of formats and
# performs a spell check.
.PHONY: all
all: documents spellcheck

# The documents target builds the document in a minimal set of formats.
.PHONY: documents
documents: main.html main_html main.epub

.PHONY: extra_documents
extra_documents: main.pdf

# The spellcheck format performs a spell check on the document.
.PHONY: spellcheck
spellcheck: spellcheck_done

# The clean target removes all files generated during the build process.
.PHONY: clean
clean:
	rm -f main.html main.pdf main.epub main.tex main.texi
	rm -rf main_html
	rm -f spellcheck_expected_sorted.txt
	rm -f missfont.log
	rm -f spellcheck_result.txt
	rm -f spellcheck_done
	rm -f main.gen.md main.tmp.md

# The install target installs the build document (in various formats)
# in the directory $(INSTALL_DIR).
.PHONY: install
install: all
	if [ ! -d "$(INSTALL_DIR)" ]; then \
	    mkdir -p "$(INSTALL_DIR)" || exit 1; \
	  fi
	for dir in html html/images html/css; do \
	    if [ ! -d "$(INSTALL_DIR)/$$dir" ]; then \
	        mkdir -p "$(INSTALL_DIR)/$$dir" || exit 1; \
	    fi; \
	  done
	#cp -f images/cpp_logo.png "$(INSTALL_DIR)/html/images"
	cp -f css/*.css "$(INSTALL_DIR)/html/css"
	cp -f main.html "$(INSTALL_DIR)/html/index.html"
	cp -r -f main_html "$(INSTALL_DIR)/html_split"
	cp -f main.epub "$(INSTALL_DIR)"
	if [ -f main.pdf ]; then \
	      cp -f main.pdf "$(INSTALL_DIR)"; \
	    fi

################################################################################
# Some additional configuration.
################################################################################

SPELLCHECK_DIR = $(TOP_DIR)/etc
ASPELL_FRONTEND = ../bin/aspell_frontend
COMPARE_WORD_LISTS = ../bin/compare_word_lists

################################################################################
# Establish Pandoc settings.
################################################################################

PREPROC_OPTIONS += --variable version="$(DOC_VERSION)"
PREPROC_OPTIONS += --variable ghp_name="$(GHP_NAME)"
PREPROC_OPTIONS += --variable doc_url="https://mdadams.github.io/signal-processing-educational-resources"
PREPROC_OPTIONS += --variable git_repo_url="https://github.com/mdadams/signal-processing-educational-resources.git"

PANDOC_OPTIONS += --toc --toc-depth 3
PANDOC_OPTIONS += --number-sections
PANDOC_OPTIONS += --standalone
PANDOC_OPTIONS += --pdf-engine=xelatex
#PANDOC_OPTIONS += --lua-filter=pandoc/filters/meta_vars.lua
#PANDOC_OPTIONS += --metadata date="`date +%Y-%m-%d`"
PANDOC_OPTIONS += --metadata version=$(DOC_VERSION)
#PANDOC_OPTIONS += --bibliography biblio.bib
#PANDOC_OPTIONS += --filter=pandoc-citeproc
#PANDOC_OPTIONS += --citeproc --csl chicago-fullnote-bibliography.csl CITATIONS
#PANDOC_OPTIONS += --css $(CSS_FILE)

INPUT_FORMAT = markdown+header_attributes+multiline_tables

#CSS_FILE = $(TOP_DIR)/src/css/default-pandoc.css
EPUB_CSS_FILE = $(TOP_DIR)/src/css/default-pandoc.css

#HTML_TEMPLATE = $(TOP_DIR)/pandoc/templates/bootstrap_menu.html
HTML_TEMPLATE = $(TOP_DIR)/pandoc/templates/uikit.html

################################################################################
# Rules for generating the document in various formats.
################################################################################

PANDOC_PREPROCESSOR = $(TOP_DIR)/bin/preprocess

main.gen.md: main.md
	rm -f main.gen.md
	$(PANDOC_PREPROCESSOR) < $< > main.tmp.md
	pandoc $(PREPROC_OPTIONS) --template main.tmp.md main.tmp.md > main.gen.md

main.html: main.gen.md $(SOURCES)
	pandoc $(PANDOC_OPTIONS) --template $(HTML_TEMPLATE) --from $(INPUT_FORMAT) -o $@ main.gen.md
	#pandoc $(PANDOC_OPTIONS) --template $(HTML_TEMPLATE) --css $(CSS_FILE) --from $(INPUT_FORMAT) -o $@ main.md

main.epub: main.gen.md $(SOURCES)
	pandoc $(PANDOC_OPTIONS) -c $(EPUB_CSS_FILE) --from $(INPUT_FORMAT) -o $@ main.gen.md

main.pdf: main.gen.md $(SOURCES)
	pandoc $(PANDOC_OPTIONS) --from $(INPUT_FORMAT) -o $@ main.gen.md

main.docbook: main.gen.md $(SOURCES)
	pandoc $(PANDOC_OPTIONS) --from $(INPUT_FORMAT) -o $@ main.gen.md

main.tex: main.gen.md $(SOURCES)
	pandoc $(PANDOC_OPTIONS) --from $(INPUT_FORMAT) --to latex -o $@ main.gen.md

main_html: $(SOURCES) main.texi
	makeinfo --no-validate --force --html -o main_html main.texi

main.texi: main.gen.md $(SOURCES) $(GENERATED_MARKDOWN)
	pandoc $(PANDOC_OPTIONS) --from $(INPUT_FORMAT) -o $@ main.gen.md

################################################################################
# Rule for spellchecking.
################################################################################

spellcheck_done: main.html
	rm -f spellcheck_result.txt 
	PATH="$(TOP_DIR)/bin:$$PATH" pandoc --from $(INPUT_FORMAT) \
	  --lua-filter $(TOP_DIR)/pandoc/filters/spellcheck.lua \
	  main.md | sort | uniq > spellcheck_result.txt
	status=0; \
	  $(COMPARE_WORD_LISTS) -q \
	    $(SPELLCHECK_DIR)/ignored_words.txt spellcheck_result.txt || status=1; \
	  if [ $$status -ne 0 ]; then \
	      echo "SPELLING ERRORS DETECTED:"; \
	      $(COMPARE_WORD_LISTS) \
	        $(SPELLCHECK_DIR)/ignored_words.txt spellcheck_result.txt; \
	      if [ $(DOC_SPELLCHECK_MUST_PASS) -ne 0 ]; then \
	          echo "ERROR: spelling errors detected"; \
	          exit 1; \
	      else \
	          echo "WARNING: spelling errors detected"; \
	          exit 0; \
	      fi; \
	  fi; \
	  touch spellcheck_done

################################################################################
################################################################################
