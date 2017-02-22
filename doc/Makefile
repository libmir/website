###########################################################
# This file builds the mir documentation.
###########################################################

# tags
LATEST:=$(git describe --abbrev=0 --tags | tr -d v)

# binaries
DMD=dmd
RDMD=rdmd
DDOC=$(DMD) -w -c -o- -version=StdDdoc

# folders
DOC_OUTPUT_DIR=web
DOC_SOURCE_DIR=doc
GENERATED=.generated
DLANGORG_DIR=$(DOC_SOURCE_DIR)/dlang.org
ARTWORK_DIR=$(DOC_SOURCE_DIR)/artwork

ALGORITHM_DIR=$(DOC_SOURCE_DIR)/../all
MIR_REPOS=$(addsuffix /source, $(addprefix $(DOC_SOURCE_DIR)/../, \
	mir-algorithm mir-random mir-math mir-internal mir-cpuid mir-glas \
))

###########################################################
# setup packages
##########################################################

# Packages in mir. Just mention the package name here. The contents of package
# xy/zz is in variable PACKAGE_xy_zz. This allows automation in iterating
# packages and their modules.
TLD_PACKAGES = mir cpuid glas
MIR_PACKAGES = $(TLD_PACKAGES) $(addprefix mir/, \
		ndslice random random/engine random/flex math math/func \
)

PACKAGE_mir = conv functional primitives utility

PACKAGE_mir_ndslice = package algorithm allocation dynamic field iterator package slice sorting stack topology

PACKAGE_mir_math = sum constant
PACKAGE_mir_math_func = expdigamma

PACKAGE_mir_random = package algorithm variable
PACKAGE_mir_random_engine = package linear_congruential mersenne_twister xorshift
PACKAGE_mir_random_flex = package

PACKAGE_cpuid = unified common x86_any intel amd

PACKAGE_glas = ndslice fortran

MOD_EXCLUDES=$(addprefix --ex=, \
	mir.ndslice.internal \
	mir.math.internal \
	mir.internal. \
	mir.random.flex.internal. \
	glas.internal. glas.precompiled. \
)

###########################################################
# Setup macros + generate dynamic info needed
###########################################################

all: html

DLANGORG_MACROS=$(addprefix $(DLANGORG_DIR)/, macros html dlang.org)
STDDOC=$(addsuffix .ddoc, ${DLANGORG_MACROS} ${GENERATED}/${LATEST} $(DLANGORG_DIR)/std $(DOC_SOURCE_DIR)/custom ${GENERATED}/mir) $(NODATETIME)

${GENERATED}/${LATEST}.ddoc :
	mkdir -p $(dir $@)
	echo "LATEST=${LATEST}" >$@

${GENERATED}/mir.ddoc : $(DOC_SOURCE_DIR)/modlist.d $(MIR_REPOS)
	mkdir -p $(dir $@)
	$(RDMD) --compiler=$(DMD) $< $(MIR_REPOS) $(MOD_EXCLUDES) $(addprefix --dump=, $(TLD_PACKAGES)) >$@

###########################################################
# Makefile bootstrapping
# It's mostly about the conversion from mir.foo -> mir_foo
###########################################################

# Given one or more packages, returns the modules they contain
P2MODULES=$(foreach P,$1,$(addprefix $P/,$(PACKAGE_$(subst /,_,$P))))
MIR_MODULES=$(call P2MODULES,$(MIR_PACKAGES))
SRC_DOCUMENTABLES = index.d $(addsuffix .d,$(MIR_MODULES))

# D file to html, e.g. mir/combinatorics -> mir_combinatorics.html
# But "package.d" is special cased: std/range/package.d -> std_range.html
D2HTML=$(foreach p,$1,$(if $(subst package.d,,$(notdir $p)),$(subst /,_,$(subst .d,.html,$p)),$(subst /,_,$(subst /package.d,.html,$p))))

HTMLS=$(addprefix $(DOC_OUTPUT_DIR)/, \
	$(call D2HTML, $(SRC_DOCUMENTABLES)))

$(DOC_OUTPUT_DIR)/. :
	mkdir -p $@

# everything except index.d needs a source path
ADDSOURCE=$(if $(subst index.d,,$1),$(ALGORITHM_DIR)/$1,$1)

# For each module, define a rule e.g.:
# ../web/phobos/std_conv.html : std/conv.d $(STDDOC) ; ...
$(foreach p,$(SRC_DOCUMENTABLES),$(eval \
$(DOC_OUTPUT_DIR)/$(call D2HTML,$p) : $(call ADDSOURCE,$p) $(STDDOC) ;\
  $(DDOC) $(STDDOC) $(addprefix -I, $(MIR_REPOS)) -Df$$@ $$<))

###########################################################
# Setup all other resources needed by dlang.org
###########################################################

IMAGES=images/mir.svg favicon.ico

JAVASCRIPT=$(addsuffix .js, $(addprefix js/, \
	dlang ddox listanchors run run-main-website jquery-1.7.2.min))

STYLES=$(addsuffix .css, $(addprefix css/, \
	style print custom ))

ALL_FILES = $(addprefix $(DOC_OUTPUT_DIR)/, \
	$(STYLES) $(IMAGES) $(JAVASCRIPT))

$(DOC_OUTPUT_DIR)/css/custom.css: $(DOC_SOURCE_DIR)/custom.css
	@mkdir -p $(dir $@)
	cp $< $@

$(DOC_OUTPUT_DIR)/images/mir.svg: $(ARTWORK_DIR)/logo/mir_site_logo.svg
	@mkdir -p $(dir $@)
	cp $< $@

$(DOC_OUTPUT_DIR)/% : $(DLANGORG_DIR)/%
	@mkdir -p $(dir $@)
	cp $< $@

html : $(DOC_OUTPUT_DIR)/. $(HTMLS) $(ALL_FILES)

clean:
	rm -rf $(DOC_OUTPUT_DIR)
	rm -rf $(GENERATED)

# prints the listed modules and sources
debug:
	@echo $(SRC_DOCUMENTABLES)
	@echo $(STDDOC)
