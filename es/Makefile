# This makefile requires GNU make.

sources := \
	00book.tex \
	99book.bib \
	99defs.tex \
	build_id.tex \
	branch.tex \
	cmdref.tex \
	collab.tex \
	concepts.tex \
	daily.tex \
	filenames.tex \
	hg_id.tex \
	hgext.tex \
	hook.tex \
	intro.tex \
	mq.tex \
	mq-collab.tex \
	mq-ref.tex \
	preface.tex \
	srcinstall.tex \
	template.tex \
	tour-basic.tex \
	tour-merge.tex \
	undo.tex

image-sources := \
	feature-branches.dot \
	filelog.svg \
	kdiff3.png \
	metadata.svg \
	mq-stack.svg \
	note.png \
	revlog.svg \
	snapshot.svg \
	tour-history.svg \
	tour-merge-conflict.svg \
	tour-merge-merge.svg \
	tour-merge-pull.svg \
	tour-merge-sep-repos.svg \
	undo-manual.dot \
	undo-manual-merge.dot \
	undo-non-tip.dot \
	undo-simple.dot \
	wdir.svg \
	wdir-after-commit.svg \
	wdir-branch.svg \
	wdir-merge.svg \
	wdir-pre-branch.svg

image-dot := $(filter %.dot,$(image-sources))
image-svg := $(filter %.svg,$(image-sources))
image-png := $(filter %.png,$(image-sources))

image-pdf := $(image-dot:%.dot=%.pdf) $(image-svg:%.svg=%.pdf) $(image-png)
image-html := $(image-dot:%.dot=%.png) $(image-svg:%.svg=%.png) $(image-png)

example-sources := \
	backout \
	bisect \
	branching \
	branch-named \
	branch-repo \
	cmdref \
	daily.copy \
	daily.files \
	daily.rename \
	daily.revert \
	extdiff \
	filenames \
	hook.msglen \
	hook.simple \
	hook.ws \
	issue29 \
	mq.guards \
	mq.qinit-help \
	mq.dodiff \
	mq.id \
	mq.tarball \
	mq.tools \
	mq.tutorial \
	rename.divergent \
	rollback \
	tag \
	template.simple \
	template.svnstyle \
	tour \
	tour-merge-conflict

example-prereqs := \
	/usr/bin/merge

dist-sources := \
	../html/hgicon.png \
	../html/index.html.var \
	../html/index.en.html \
	../html/index.es.html

latex-options = \
	-interaction batchmode \
	-output-directory $(dir $(1)) \
	-jobname $(basename $(notdir $(1)))

hg = $(shell which hg)

hg-id = $(shell hg parents --template '{node|short}, fechado {date|isodate},\n')

hg-version = $(shell hg version -q | \
		     sed 's,.*(versión \(unknown\|[a-f0-9+]*\)),\1,')

all: pdf html

pdf: pdf/hgbook.pdf

define pdf
	mkdir -p $(dir $@)
	TEXINPUTS=$(dir $<): pdflatex $(call latex-options,$@) $< || (rm -f $@; exit 1)
	cp 99book.bib $(dir $@)
	cd $(dir $@) && bibtex $(basename $(notdir $@))
	cd $(dir $@) && makeindex $(basename $(notdir $@))
	TEXINPUTS=$(dir $<): pdflatex $(call latex-options,$@) $< || (rm -f $@; exit 1)
	TEXINPUTS=$(dir $<): pdflatex $(call latex-options,$@) $< || (rm -f $@; exit 1)
	if grep 'Reference.*undefined' $(@:.pdf=.log); then exit 1; fi
endef

pdf/hgbook.pdf: $(sources) examples $(image-pdf)
	$(call pdf)

html: onepage split

onepage: $(htlatex) html/onepage/hgbook.html html/onepage/hgbook.css $(image-html:%=html/onepage/%)

html/onepage/%: %
	cp $< $@

split: $(htlatex) html/split/hgbook.html html/split/hgbook.css $(image-html:%=html/split/%)

html/split/%: %
	cp $< $@

# This is a horrible hack to work around the fact that the htlatex
# command in tex4ht is itself a horrible hack.  I really don't want to
# include verbatim the big wad of TeX that is repeated in that script,
# but I've given up and run a hacked copy as htlatex.book here.

define htlatex
	mkdir -p $(dir $(1))
	cp 99book.bib $(dir $(1))
	TEXINPUTS=$(dir $(2)): ./htlatex.book $(2) "bookhtml,html4-uni,$(3)" " -cunihtf -utf8" "$(dir $(1))" "$(call latex-options,$(1))" || (rm -f $(1); exit 1)
	cd $(dir $(1)) && tex4ht -f/$(basename $(notdir $(1))) -cvalidate -cunihtf
	cd $(dir $(1)) && t4ht -f/$(basename $(notdir $(1)))
	./fixhtml.py $(dir $(1))/*.html
	rm $(dir $(1))/hgbook.css
endef

html/onepage/hgbook.html: $(sources) examples $(image-html) bookhtml.cfg
	$(call htlatex,$@,$<)

html/split/hgbook.html: $(sources) examples bookhtml.cfg
	$(call htlatex,$@,$<,2)

# Produce 90dpi PNGs for the web.

%.png: %.svg fixsvg
	./fixsvg $<
	inkscape -D -e $@ $<-tmp.svg
	rm $<-tmp.svg

%.svg: %.dot
	dot -Tsvg -o $@ $<

# Produce eps & pdf for the pdf

%.pdf: %.eps
	epstopdf $<

%.eps: %.svg
	./fixsvg $<
	inkscape -E $@ $<-tmp.svg
	rm $<-tmp.svg

%.eps: %.dot
	dot -Tps -o $@ $<

examples: $(example-prereqs) examples/.run

examples/.run: $(example-sources:%=examples/%.run)
	touch examples/.run

examples/%.run: examples/% examples/run-example
	cd examples && ./run-example $(notdir $<)

changelog := $(wildcard ../.hg/store/00changelog.[id])
ifeq ($(changelog),)
changelog := $(wildcard ../.hg/00changelog.[id])
endif

build_id.tex: $(changelog)
	echo -n '$(hg-id)' > build_id.tex

hg_id.tex: $(hg)
	echo -n '$(hg-version)' > hg_id.tex

clean:
	rm -rf dist html pdf \
		$(image-dot:%.dot=%.pdf) \
		$(image-dot:%.dot=%.png) \
		$(image-svg:%.svg=%.pdf) \
		$(image-svg:%.svg=%.png) \
		examples/*.{lxo,run} examples/.run build_id.tex hg_id.tex

install: pdf split $(dist-sources) 
	rm -rf dist
	mkdir -p dist
	cp pdf/hgbook.pdf dist
	cp html/split/*.{css,html,png} dist
	cp html/onepage/hgbook.html dist/onepage.html
	ln -s index.es.html dist/index.html
	cp $(dist-sources) dist

rsync: install
	rsync -avz --delete dist/ ikks@sulaco.devnull.li:public_html/hgbook/
