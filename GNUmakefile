# (c) 2020 Ezequiel Valenzuela <evp.dev.ezequiel.p.01@jev.me.uk>
# Licensed under GNU Lesser General Public License v3.0 (or later, at your option)

TARGETS_PHONY_RECURSIVE := \
 clean \
 # end

TARGETS_PHONY := \
 all \
 $(TARGETS_PHONY_RECURSIVE) \
 help \
 files \
 env \
 dockercomposeyml \
 compose-up up \
 compose-stop stop \
 compose-rm docker-clean \
 recreate-up recreate \
 \
 submake_std_builddep \
 # end

# this has an effect on the '+=' used later
# (see: 'info make', section "3.7 How 'make' Reads a Makefile",
# subsection "Variable Assignment"):
#
#     For the append operator, '+=', the right-hand side is considered
#  immediate if the variable was previously set as a simple variable (':='
#  or '::='), and deferred otherwise.
#
TARGETS_FILES :=
#? unused: ALL_SUBMAKE_STD :=
ALL_SUBMAKE_STD_VARNAME_PREFS :=
ALL_SUBMAKE_STD_EXTRADEPS :=
ALL_TAGIDS_TAGMAPPING :=

# other variables that we'll just react to, but we don't define for the current
# makefile:
#  * MAK_THIS_ISDOCKERCOMPOSEMAKE_SUBMAKE
#     (presence flag)
#     if non-empty, this invocation is not the "dockercomposemake top-level"

MAK_THIS_MAKEFILE := $(lastword $(MAKEFILE_LIST))
ifeq ($(MAK_MAIN_MAKEFILE),)
 MAK_MAIN_MAKEFILE := $(MAK_THIS_MAKEFILE)
 .PRECIOUS: $(MAK_MAIN_MAKEFILE)
endif
MAK_MAIN_MAKEFILE_FORSUBMAKE := $(realpath $(MAK_MAIN_MAKEFILE))

SRC_FILES_COMMON := $(MAK_THIS_MAKEFILE)

all: compose-up

# done: create a function to return a list without duplicated elements, and maintaining the same order.
#  done: and have lists like the "hosts" lists de-duplicated using this mechanism (to avoid duplicated wildcard/file matching)
#  IDEA: MAYBE: use some of the techniques in
#  'dockercomposemake/gnumake-utils.d/GNUmakefile-util-maptocustomid_extractfromfname'
#   ref: ifeq ($(filter $(MAK_TOOL_CUR_MAPTO),$(ALL_TAGIDS_ALL)),) ...

define FUNC_STRTOESCAPEDPATTERN
$(subst $(PERCENTSYMBOL),$(TOKEN_PERCENTSYMBOL),$(1))
endef

define FUNC_ESCAPEDPATTERNTOSTR
$(subst $(TOKEN_PERCENTSYMBOL),$(PERCENTSYMBOL),$(1))
endef

#+ prev: v1: define FUNC_WORDLIST_REMOVEDUPLICATES
#+ prev: v1: $(strip \
#+ prev: v1: )$(eval undefine T_WORDLIST_REMOVEDUPLICATES_OUTLIST)$(strip \
#+ prev: v1: )$(foreach w,$(strip \
#+ prev: v1:  )$(call FUNC_STRTOESCAPEDPATTERN,$(1)),$(strip \
#+ prev: v1:  )$(if $(strip \
#+ prev: v1:   )$(filter $(w),$(T_WORDLIST_REMOVEDUPLICATES_OUTLIST)),$(strip \
#+ prev: v1:   ),$(strip \
#+ prev: v1:   )$(eval $(strip \
#+ prev: v1:    )T_WORDLIST_REMOVEDUPLICATES_OUTLIST += $(w)$(strip \
#+ prev: v1:   ))$(strip \
#+ prev: v1:  ))$(strip \
#+ prev: v1: ))$(strip \
#+ prev: v1: )$(strip $(strip \
#+ prev: v1:  )$(call FUNC_ESCAPEDPATTERNTOSTR,$(strip \
#+ prev: v1:   )$(T_WORDLIST_REMOVEDUPLICATES_OUTLIST)$(strip \
#+ prev: v1:  ))$(strip \
#+ prev: v1: ))$(strip \
#+ prev: v1: )
#+ prev: v1: endef
#+ prev: v2: define FUNC_WORDLIST_REMOVEDUPLICATES
#+ prev: v2: $(strip $(strip \
#+ prev: v2:  )$(eval undefine T_WORDLIST_REMOVEDUPLICATES_OUTLIST)$(strip \
#+ prev: v2:  )$(foreach w,$(strip \
#+ prev: v2:   )$(call FUNC_STRTOESCAPEDPATTERN,$(1)),$(strip \
#+ prev: v2:   )$(if $(strip \
#+ prev: v2:    )$(filter $(w),$(T_WORDLIST_REMOVEDUPLICATES_OUTLIST)),$(strip \
#+ prev: v2:    ),$(strip \
#+ prev: v2:    )$(eval $(strip \
#+ prev: v2:     )T_WORDLIST_REMOVEDUPLICATES_OUTLIST += $(w)$(strip \
#+ prev: v2:    ))$(strip \
#+ prev: v2:   ))$(strip \
#+ prev: v2:  ))$(strip \
#+ prev: v2:  )$(call FUNC_ESCAPEDPATTERNTOSTR,$(strip \
#+ prev: v2:   )$(T_WORDLIST_REMOVEDUPLICATES_OUTLIST)$(strip \
#+ prev: v2:  ))$(strip \
#+ prev: v2: ))
#+ prev: v2: endef
#
# args:
# $(1): word to be added to the front (can be initially empty);
# $(2): list tail;;
define FUNC_WORDLIST_REMOVEDUPLICATES_WORDANDTAILESCAPED
$(strip $(strip \
 )$(strip $(1)) $(strip \
 )$(filter-out $(strip \
  )$(strip $(1)),$(strip \
  )$(if $(strip $(2)),$(strip \
   )$(call FUNC_WORDLIST_REMOVEDUPLICATES_WORDANDTAILESCAPED,$(strip \
    )$(firstword $(strip $(2))),$(strip \
    )$(wordlist 2,9999,$(strip $(2)))$(strip \
   ))$(strip \
  ))$(strip \
 ))$(strip \
))$(strip \
)
endef
#
# args:
# $(1): the word list
#
# notes:
#   * the returned string has the same words as the input, but it might not
#     have the same number of whitespaces (it might have been '$(strip )'-ed
#     when being processed).
#
#? $(call FUNC_WORDLIST_REMOVEDUPLICATES_WORDANDTAILESCAPED,,$(strip $(2)))
define FUNC_WORDLIST_REMOVEDUPLICATES
$(strip $(strip \
 )$(call FUNC_ESCAPEDPATTERNTOSTR,$(strip \
  )$(call FUNC_WORDLIST_REMOVEDUPLICATES_WORDANDTAILESCAPED,,$(strip \
   )$(call FUNC_STRTOESCAPEDPATTERN,$(strip \
    )$(strip $(1))$(strip \
   ))$(strip \
  ))$(strip \
 ))$(strip \
))$(strip \
)
endef

ifeq ($(origin HOSTNAMES_ALL_ORIG),undefined)
 # short flags: -f -s -a -A
 HOSTNAMES_ALL_ORIG := $(call FUNC_WORDLIST_REMOVEDUPLICATES,$(strip \
   )$(HOSTNAME) $(strip \
   )$(foreach flg,--long --short --alias --all-fqdns,$(strip \
    )$(shell hostname $(flg))$(strip \
   ))$(strip \
  ))
 export HOSTNAMES_ALL_ORIG
endif

define HOSTNAME_PROC_EXTRACTHOSTPART
$(foreach hnow,$(1),$(firstword $(subst ., ,$(hnow))))
endef

define HOSTNAME_PROC_GETHOSTSWITHOUTDOMAIN
$(filter $(call HOSTNAME_PROC_EXTRACTHOSTPART,$(1)),$(1))
endef

define HOSTNAME_PROC_HOSTNAMESDOTSTOHYPHENS
$(subst .,-,$(1))
endef

define HOSTNAME_PROC_HOSTNAMESDOTSTOUNDERSCORES
$(subst .,_,$(1))
endef

define HOSTNAME_PROC_IDENTITY
$(1)
endef

# disabled/removed:
# * because we are using the '.' to allow for multiple "parts" using the same
#   "basename":
#   HOSTNAME_PROC_IDENTITY \
#
HOSTNAMES_ALL := $(call FUNC_WORDLIST_REMOVEDUPLICATES,$(strip \
  )$(foreach fnow, \
    HOSTNAME_PROC_EXTRACTHOSTPART \
    HOSTNAME_PROC_GETHOSTSWITHOUTDOMAIN \
    HOSTNAME_PROC_HOSTNAMESDOTSTOHYPHENS \
    HOSTNAME_PROC_HOSTNAMESDOTSTOUNDERSCORES \
   ,$(strip \
   )$(call $(fnow),$(HOSTNAMES_ALL_ORIG)) $(strip \
  ))$(strip \
 ))

SPACE := $(subst x, ,x)
TAB := $(subst x,	,x)

define NEWLINE


endef

PERCENTSYMBOL := %
COMMA := ,

# args:
# $(1): list of words to make TOKEN_{word} values for
#-? v1: define EXPR_MAKE_TOKENS
#-? v1: $$(foreach tokenid,$$(strip $(1)),$$(strip \
#-? v1:  TOKEN_$$(tokenid) := <<$$(tokenid)>> \
#-? v1: )$$(NEWLINE))
#-? v1: endef
#-? v2: define EXPR_MAKE_TOKENS
#-? v2: $$(subst _temp_newline_,$$(NEWLINE),$(strip \
#-? v2: $$(foreach tokenid,$(1),$$(info DEBUG: TOKEN_$$(tokenid) = <<$$(tokenid)>>)_temp_newline_ ) \
#-? v2: ))
#-? v2: endef
define EVAL_MAKE_TOKENS
$(foreach tokenid,$(strip $(1)),$(eval $(strip \
 TOKEN_$$(tokenid) := <<$$(tokenid)>> \
)))
endef

# MAYBE: nice (and a non-trivial implementation) idea:
#  store all the generated tokens in a list, and make the new functions:
#   * FUNC_TOKENISE_ALL(string);
#     'word1 word2' -> 'word1<<space>>word2'
#   * FUNC_EXPANDTOKENS_ALL(string);
#     (transforms it back to its original form)
# prev: v2: #? $(eval $(call EXPR_MAKE_TOKENS,SPACE NEWLINE))
# prev: v2: ifneq ($(VERBOSE),)
# prev: v2: # v1: $(info DEBUG: '$(call EXPR_MAKE_TOKENS,SPACE NEWLINE)')
# prev: v2: endif
# v1: #$(eval         $(call EXPR_MAKE_TOKENS,SPACE NEWLINE))
$(call EVAL_MAKE_TOKENS,SPACE TAB NEWLINE)
#+ $(eval $(foreach tokenid,SPACE NEWLINE,TOKEN_$(tokenid) := __$(tokenid)__$(NEWLINE)) )

$(call EVAL_MAKE_TOKENS,PERCENTSYMBOL)

$(call EVAL_MAKE_TOKENS,FIELDSEP NULL)

#+ v1: define FUNC_WORDS_SORTBYPARTNUMBER
#+ v1: $(foreach e,$(strip \
#+ v1:  )$(sort $(strip \
#+ v1:   )$(foreach w,$(strip $(1)),$(strip \
#+ v1:    )$(firstword $(strip $(strip \
#+ v1:     )$(foreach pref,$(strip $(2)),$(strip \
#+ v1:      )$(if $(findstring $(pref),$(w)),$(strip \
#+ v1:       )$(lastword $(subst $(pref),$(SPACE),$(w)))$(TOKEN_FIELDSEP)$(w)$(strip \
#+ v1:      )) $(strip \
#+ v1:     )) $(strip \
#+ v1:     )$(3)$(TOKEN_FIELDSEP)$(w)$(strip \
#+ v1:    )))$(strip \
#+ v1:   ))$(strip \
#+ v1:  )),$(strip \
#+ v1:  )$(lastword $(subst $(TOKEN_FIELDSEP),$(SPACE),$(e)))$(strip \
#+ v1: ))
#+ v1: endef
#+ v2: define FUNC_WORDS_SORTBYPARTNUMBER
#+ v2: $(foreach e,$(strip \
#+ v2:  )$(sort $(strip \
#+ v2:   )$(foreach w,$(strip $(1)),$(strip \
#+ v2:    )$(firstword $(strip $(strip \
#+ v2:     )$(foreach pref,$(strip $(2)),$(strip \
#+ v2:      )$(if $(findstring $(pref),$(w)),$(strip \
#+ v2:       )$(subst $(SPACE),$(pref),$(strip \
#+ v2:        )$(wordlist 2,99,$(subst $(pref),$(SPACE),$(w)))$(strip \
#+ v2:       ))$(TOKEN_FIELDSEP)$(w)$(strip \
#+ v2:      )) $(strip \
#+ v2:     )) $(strip \
#+ v2:     )$(3)$(TOKEN_FIELDSEP)$(w)$(strip \
#+ v2:    )))$(strip \
#+ v2:   ))$(strip \
#+ v2:  )),$(strip \
#+ v2:  )$(lastword $(subst $(TOKEN_FIELDSEP),$(SPACE),$(e)))$(strip \
#+ v2: ))
#+ v2: endef
#+ v3: define FUNC_WORDS_SORTBYPARTNUMBER
#+ v3: $(foreach e,$(strip \
#+ v3:  )$(sort $(strip \
#+ v3:   )$(foreach wo,$(strip $(1)),$(subst $(SPACE),,$(strip \
#+ v3:    )$(foreach w,$(strip \
#+ v3:     )$(if $(findstring b,$(strip $(4))),$(strip \
#+ v3:       )$(lastword $(strip \
#+ v3:        )$(subst /,$(SPACE),$(wo))$(strip \
#+ v3:       )),$(strip \
#+ v3:       )$(wo)$(strip \
#+ v3:      )),$(strip \
#+ v3:     )$(firstword $(strip $(strip \
#+ v3:      )$(foreach pref,$(strip $(2)),$(strip \
#+ v3:       )$(if $(findstring $(pref),$(w)),$(strip \
#+ v3:        )$(subst $(SPACE),$(pref),$(strip \
#+ v3:         )$(wordlist 2,99,$(subst $(pref),$(SPACE),$(w)))$(strip \
#+ v3:        ))$(TOKEN_FIELDSEP)$(w)$(strip \
#+ v3:       )) $(strip \
#+ v3:      )) $(strip \
#+ v3:      )$(3)$(TOKEN_FIELDSEP)$(w)$(strip \
#+ v3:     )))$(strip \
#+ v3:    ))$(strip \
#+ v3:   )$(TOKEN_FIELDSEP)$(wo)$(strip \
#+ v3:  )))$(strip \
#+ v3:  )),$(strip \
#+ v3:  )$(lastword $(subst $(TOKEN_FIELDSEP),$(SPACE),$(e)))$(strip \
#+ v3: ))
#+ v3: endef
#
# LATER: TODO: check that it works correclty with words containing the prefix more than once.
# args:
# $(1): words (whitespace separted);
# $(2): string(s) just before the part number (can this be empty?);
# $(3): sort-friendly prefix string to use for words without a part number
#       (default: ?);
# $(4): [optional] flags
#       'b': use basenames only to determine the sorting order;
# MAYBE: support suffix-only (like '-', '_') to have files like:
#  01-part_one
#  somedir/02-part_two
# MAYBE: have an argument to work on either the basename or the full name
#  (possibly only relevant when using suffix-only, but it could be useful when
#  specifying words that are pathnames that might include one of the allowed
#  prefixes in the directory name)
# TESTING: add after the last '$(lastword ...$(e))...' line:
# )$(TOKEN_FIELDSEP)orig:$(TOKEN_FIELDSEP)$(e)$(strip \
#
define FUNC_WORDS_SORTBYPARTNUMBER
$(foreach e,$(strip \
 )$(sort $(strip \
  )$(foreach wo,$(strip $(1)),$(subst $(SPACE),,$(strip \
   )$(foreach w,$(strip \
    )$(if $(findstring b,$(strip $(4))),$(strip \
      )$(lastword $(strip \
       )$(subst /,$(SPACE),$(wo))$(strip \
      )),$(strip \
      )$(wo)$(strip \
     )),$(strip \
    )$(firstword $(strip $(strip \
     )$(foreach pref,$(strip $(2)),$(strip \
      )$(if $(findstring $(pref),$(w)),$(strip \
       )$(subst $(SPACE),$(pref),$(strip \
        )$(wordlist 2,99,$(subst $(pref),$(SPACE),$(w)))$(strip \
       ))$(TOKEN_FIELDSEP)$(w)$(strip \
      )) $(strip \
     )) $(strip \
     )$(3)$(TOKEN_FIELDSEP)$(w)$(strip \
    )))$(strip \
   ))$(strip \
  )$(TOKEN_FIELDSEP)$(wo)$(strip \
 )))$(strip \
 )),$(strip \
 )$(lastword $(subst $(TOKEN_FIELDSEP),$(SPACE),$(e)))$(strip \
))
endef

ifneq ($(filter 1,$(TESTING)),)
test_words = \
 prefix_common_no_part \
 prefix_common.part-03_three_tres.cfg \
 zzz.part-02_due.part-20_twenty.cfg \
 somedir/prefix_another.part_10_ten_diez.cfg \
 prefix_common.part-30_thirty_three.cfg \
 somebasedir.part_05_somedirsuff/prefix_another.part_22_twenty_two.cfg \
 prefix_common.part-01_one_uno.cfg \
 # end
$(info TESTING: part-sorting test: orig: '$(test_words)'; sorted: '$(call FUNC_WORDS_SORTBYPARTNUMBER,$(test_words),.part- .part_,99,b)';)
endif

#? # args:
#? # $(1): parameter normally sent to '$(wildcard )';
#? # $(2): [optional] $(TOKEN_NULL)-aware directory prefix;
#? define FUNC_WILDCARDNULLAWARE
#? $(wildcard $(subst $(TOKEN_NULL),,$(strip $(2))) ... )
#? endef

# $(1): $(TOKEN_NULL)-aware directory prefix;
define FUNC_GETDIRFORPREF_MAYBEEMPTY
$(subst $(TOKEN_NULL),,$(strip $(1)))
endef

# $(1): $(TOKEN_NULL)-aware directory prefix;
#- define FUNC_GETDIRPREFANDSEPARATOR
#- $(strip \
#- )$(filter-out . ./,$(strip \
#-  )$(patsubst ./%,%,$(strip \
#-   )$(patsubst %/,%,$(strip \
#-    )$(subst //,/,$(strip \
#-     )$(call FUNC_GETDIRFORPREF_MAYBEEMPTY,$(1))$(strip \
#-     )$(if $(call FUNC_GETDIRFORPREF_MAYBEEMPTY,$(1)),/)$(strip \
#-    ))$(strip \
#-   ))$(strip \
#-  ))$(strip \
#- ))$(strip \
#- )
#- endef
define FUNC_GETDIRPREFANDSEPARATOR
$(strip \
)$(filter-out . ./,$(strip \
 )$(patsubst ./%,%,$(strip \
  )$(subst //,/,$(strip \
   )$(call FUNC_GETDIRFORPREF_MAYBEEMPTY,$(1))$(strip \
   )$(if $(call FUNC_GETDIRFORPREF_MAYBEEMPTY,$(1)),/)$(strip \
  ))$(strip \
 ))$(strip \
))$(strip \
)
endef

# args:
# $(1): filename prefixes: each word is a "basename" for which all possible
#       names are to be '$(wildcard )'-ed.  Only the files for the first
#       file-matching word are returned.
# $(2): [optional] list of directories in which to search for additional files
define FUNC_GETALLFILESFORALLBASENAMES_ONETOKWORDPERBASENAME
$(strip $(strip \
 )$(foreach bnow,$(strip $(1)),$(strip $(strip \
   )$(subst $(SPACE),$(TOKEN_SPACE),$(strip \
    $(foreach t_dir_orig,$(sort $(strip \
       )$(TOKEN_NULL) $(strip \
       )$(strip $(2)) $(strip \
       )$(strip $(ALL_COMMONANDHOST_PARTS_DIRS)) $(strip \
      )),$(strip $(strip \
       )$(wildcard $(strip \
        )$(foreach t_wcbasename,$(strip $(strip \
          )$(bnow) $(strip \
          )$(foreach t_sufnow,$(strip $(strip \
            ).part[-_] $(strip \
           )),$(strip \
           )$(bnow)$(t_sufnow)*$(strip \
          ))$(strip \
         )),$(strip \
         )$(strip \
          )$(call FUNC_GETDIRPREFANDSEPARATOR,$(t_dir_orig))$(strip \
          )$(t_wcbasename)$(strip \
        ))$(strip \
       ))$(strip \
      ))$(strip \
     ))$(strip \
    ))$(strip \
   ))$(strip \
  ))$(strip \
 ))$(strip \
))$(strip \
)
endef

# args:
# $(1): "tokenised" (as per
#       FUNC_GETALLFILESFORALLBASENAMES_ONETOKWORDPERBASENAME) words;
#
# returns:
# whitespace-separated untokenised words;
#
# notes:
#
#   * there will be no way to identify which "tokenised" word each resulting
#     word came from.
#
#   * the returned list would keep the original word order, and it will be free
#     from duplicates (as per '$(call FUNC_WORDLIST_REMOVEDUPLICATES )').
#
define FUNC_GETFILESLISTFROMTOKWORDS
$(strip $(strip \
 )$(call FUNC_WORDLIST_REMOVEDUPLICATES,$(strip \
  )$(subst $(TOKEN_SPACE),$(SPACE),$(strip \
   )$(strip $(1))$(strip \
  ))$(strip \
 ))$(strip \
))$(strip \
)
endef

#+ v1: define FUNC_GETALLFILESFORFIRSTMATCHINGBASENAME
#+ v1: $(subst $(TOKEN_SPACE),$(SPACE),$(firstword $(strip \
#+ v1:  $(foreach bnow,$(strip $(1)),$(strip \
#+ v1:   $(subst $(SPACE),$(TOKEN_SPACE),$(strip \
#+ v1:    $(wildcard $(bnow)) \
#+ v1:    $(foreach sufnow,.part[-_],$(wildcard $(bnow)$(sufnow)*)) \
#+ v1:   )) \
#+ v1:  )) \
#+ v1: )))
#+ v1: endef
#+ v2: define FUNC_GETALLFILESFORFIRSTMATCHINGBASENAME
#+ v2: $(subst $(TOKEN_SPACE),$(SPACE),$(firstword $(strip \
#+ v2:  $(foreach bnow,$(strip $(1)),$(strip \
#+ v2:   $(subst $(SPACE),$(TOKEN_SPACE),$(strip \
#+ v2:    $(wildcard $(bnow)) \
#+ v2:    $(foreach sufnow,.part[-_],$(wildcard $(bnow)$(sufnow)*)) \
#+ v2:   )) \
#+ v2:  )) \
#+ v2: )))
#+ v3: define FUNC_GETALLFILESFORFIRSTMATCHINGBASENAME
#+ v3: $(subst $(TOKEN_SPACE),$(SPACE),$(firstword $(strip \
#+ v3:    $(foreach bnow,$(strip $(1)),$(strip \
#+ v3:      $(subst $(SPACE),$(TOKEN_SPACE),$(strip \
#+ v3:       $(foreach t_dir_orig,$(sort $(strip \
#+ v3:          )$(TOKEN_NULL) $(strip \
#+ v3:          )$(strip $(2)) $(strip \
#+ v3:          )$(strip $(ALL_COMMONANDHOST_PARTS_DIRS)) $(strip \
#+ v3:         )),$(strip $(strip \
#+ v3:          )$(wildcard $(strip \
#+ v3:           )$(foreach t_wcbasename,$(strip $(strip \
#+ v3:             )$(bnow) $(strip \
#+ v3:             )$(foreach t_sufnow,$(strip $(strip \
#+ v3:               ).part[-_] $(strip \
#+ v3:              )),$(strip \
#+ v3:              )$(bnow)$(t_sufnow)*$(strip \
#+ v3:             ))$(strip \
#+ v3:            )),$(strip \
#+ v3:            )$(strip \
#+ v3:             )$(call FUNC_GETDIRPREFANDSEPARATOR,$(t_dir_orig))$(strip \
#+ v3:             )$(t_wcbasename)$(strip \
#+ v3:           ))$(strip \
#+ v3:          ))$(strip \
#+ v3:         ))$(strip \
#+ v3:        ))$(strip \
#+ v3:       ))$(strip \
#+ v3:      ))$(strip \
#+ v3:     ))$(strip \
#+ v3:    ))$(strip \
#+ v3:   ))$(strip \
#+ v3:  ))$(strip \
#+ v3: ))$(strip \
#+ v3: )
#+ v3: endef
# ref: $$(t_vn_pref)THISHOST := $$(strip \
# ref:  $$(firstword $$(wildcard $$(patsubst %,$$(t_fn_pref)-host-%,$$(t_hostall)))))
# args:
# $(1): filename prefixes: each word is a "basename" for which all possible
#       names are to be '$(wildcard )'-ed.  Only the files for the first
#       file-matching word are returned.
# $(2): [optional] list of directories in which to search for additional files
define FUNC_GETALLFILESFORFIRSTMATCHINGBASENAME
$(strip $(strip \
 )$(call FUNC_GETFILESLISTFROMTOKWORDS,$(strip \
  )$(firstword $(strip \
   )$(call FUNC_GETALLFILESFORALLBASENAMES_ONETOKWORDPERBASENAME,$(strip \
    )$(1),$(strip \
    )$(2)$(strip \
   ))$(strip \
  ))$(strip \
 ))$(strip \
))$(strip \
)
endef

# args:
# $(1): filename prefixes: each word is a "basename" for which all possible
#       names are to be '$(wildcard )'-ed.
# $(2): [optional] list of directories in which to search for additional files
define FUNC_GETALLFILESFORALLMATCHINGBASENAMES
$(strip $(strip \
 )$(call FUNC_GETFILESLISTFROMTOKWORDS,$(strip \
  )$(call FUNC_GETALLFILESFORALLBASENAMES_ONETOKWORDPERBASENAME,$(strip \
   )$(1),$(strip \
   )$(2)$(strip \
  ))$(strip \
 ))$(strip \
))$(strip \
)
endef

# FUNC_TAGMAPPING_* macros {{{

# example src mapping:
#  b:c,e a:b,c d:z e:c
# expanding:
#  * 'a d'  ->
#             'a'
#               'b'
#                 'c'
#                 'e'
#                   'c'
#               'c'
#             'd'
#               'z'
#           -> 'a b c e c c d z' (with duplicates)
#             -> 'a b c e d z' (deduplicated, keep first)
#             -> 'a b e c d z' (deduplicated, keep last)
#

#-? ... define FUNC_TAGMAPPING_EXPANDONEMAPPING
#-? ... $(strip $(strip \
#-? ...  )$(firstword $(strip \
#-? ...   )$(subst :,$(SPACE),$(strip $(2)))$(strip \
#-? ...  ))$(strip \
#-? ...  ):$(strip \
#-? ...  )$(foreach ts,$(strip \
#-? ...   )$(subst $(COMMA),$(SPACE),$(strip \
#-? ...    )$(lastword $(strip \
#-? ...     )$(subst :,$(SPACE),$(strip $(2)))$(strip \
#-? ...    ))$(strip \
#-? ...   ))$(strip \
#-? ...   ),$(strip \
#-? ...   )$(call FUNC_TAGMAPPING_GETALLSOURCESFORTAGID,$(strip \
#-? ...    )$(1),$(strip \
#-? ...    )$(ts)$(strip \
#-? ...   ))$(strip \
#-? ...  ))$(strip \
#-? ... ))$(strip \
#-? ... )
#-? ... endef

#? prev: v1: # args:
#? prev: v1: # $(1): mapping complete list;
#? prev: v1: # $(2): tagid to "expand" using the mapping list in $(1);
#? prev: v1: #
#? prev: v1: # MAYBE: create another function to leave "most basic" (depended upon) to the end of the list:
#? prev: v1: # FUNC_WORDLIST_REMOVEDUPLICATES_KEEPLAST
#? prev: v1: #
#? prev: v1: #   * MAYBE: TODO: and rename: FUNC_WORDLIST_REMOVEDUPLICATES ->
#? prev: v1: #     FUNC_WORDLIST_REMOVEDUPLICATES_KEEPFIRST
#? prev: v1: #
#? prev: v1: define FUNC_TAGMAPPING_EXPANDTAGID
#? prev: v1: $(strip $(strip \
#? prev: v1:  )$(call FUNC_WORDLIST_REMOVEDUPLICATES,$(strip \
#? prev: v1:   )$(2) $(strip \
#? prev: v1:   )$(foreach tme,$(strip \
#? prev: v1:    )$(filter $(strip \
#? prev: v1:     )$(strip $(2)):%,$(strip \
#? prev: v1:     )$(strip $(1))$(strip \
#? prev: v1:    ))$(strip \
#? prev: v1:    ),$(strip \
#? prev: v1:    )$(foreach ts,$(strip \
#? prev: v1:     )$(subst $(COMMA),$(SPACE),$(strip \
#? prev: v1:      )$(lastword $(strip \
#? prev: v1:       )$(subst :,$(SPACE),$(strip $(tme)))$(strip \
#? prev: v1:      ))$(strip \
#? prev: v1:     ))$(strip \
#? prev: v1:     ),$(strip \
#? prev: v1:     )$(call FUNC_TAGMAPPING_EXPANDTAGID,$(strip \
#? prev: v1:      )$(1),$(strip \
#? prev: v1:      )$(ts)$(strip \
#? prev: v1:     ))$(strip \
#? prev: v1:    ))$(strip \
#? prev: v1:   ))$(strip \
#? prev: v1:  ))$(strip \
#? prev: v1: ))$(strip \
#? prev: v1: )
#? prev: v1: endef

# args:
# $(1): mapping complete list;
# $(2): tagid to "expand" using the mapping list in $(1);
#
# Returns the expanded list of tags for the $(2), but leaves duplicates in the
# result.  These are handled in our public caller function(s).
define FUNC_TAGMAPPING_EXPANDTAGID_STEP
$(strip $(strip \
 )$(2) $(strip \
 )$(foreach tme,$(strip \
  )$(filter $(strip \
   )$(strip $(2)):%,$(strip \
   )$(strip $(1))$(strip \
  ))$(strip \
  ),$(strip \
  )$(foreach ts,$(strip \
   )$(subst $(COMMA),$(SPACE),$(strip \
    )$(lastword $(strip \
     )$(subst :,$(SPACE),$(strip $(tme)))$(strip \
    ))$(strip \
   ))$(strip \
   ),$(strip \
   )$(call FUNC_TAGMAPPING_EXPANDTAGID_STEP,$(strip \
    )$(1),$(strip \
    )$(ts)$(strip \
   ))$(strip \
  ))$(strip \
 ))$(strip \
))$(strip \
)
endef

# args:
# $(1): mapping complete list;
# $(2): list of tagids to "expand" using the mapping list in $(1);
#
# MAYBE: create another function to leave "most basic" (depended upon) to the end of the list:
# FUNC_WORDLIST_REMOVEDUPLICATES_KEEPLAST
#
#   * MAYBE: TODO: and rename: FUNC_WORDLIST_REMOVEDUPLICATES ->
#     FUNC_WORDLIST_REMOVEDUPLICATES_KEEPFIRST
#
define FUNC_TAGMAPPING_EXPANDTAGIDS
$(strip $(strip \
 )$(call FUNC_WORDLIST_REMOVEDUPLICATES,$(strip \
  )$(foreach td,$(strip \
    )$(strip $(2))$(strip \
   ),$(strip \
    )$(call FUNC_TAGMAPPING_EXPANDTAGID_STEP,$(strip \
     )$(1),$(strip \
     )$(td)$(strip \
    ))$(strip \
   )$(strip \
  ))$(strip \
 ))$(strip \
))$(strip \
)
endef

# }}}

# TODO: implement this using the flags defined below (for
# EXPR_CALCFILESCOMMONANDHOST)
# TODO: detect that none of the flags that would require the 'awk' script are
# present.
# args:
# $(1): files (whitespace separated) to "cat";
# $(2): target file (single word);
# $(3): option flags; (see EXPR_CALCFILESCOMMONANDHOST for more details);
# $(4): "end-of-line" comment string;
define CMD_CATALLFILES_SMART
$(CMD_AWK) \
 -v 'g_flags=$(3)' \
 -v 'g_comment_eolstr=$(4)' \
 -- \
 ' \
function get_boolean_flag(s) { \
 return ( index( g_flags, s ) > 0 ); \
} \
function proc_on_end_of_file() { \
 if ( g_flag_addfileproccomments && ( length( g_current_filename ) > 0 ) ) { \
  print \
   r_current_filename_comments_leadingblanks \
   g_fileproc_comment_data_pref_end \
   g_current_filename \
   g_fileproc_comment_data_suff_end \
   ; \
  g_current_filename = ""; \
  r_flag_removeblanklines = 1; \
 } \
} \
BEGIN { \
 g_flag_removeblanklines = get_boolean_flag("B"); \
 g_flag_removeallcomments = get_boolean_flag("A"); \
 g_flag_isstandardscript = get_boolean_flag("s"); \
 g_flag_addfileproccomments = get_boolean_flag("i"); \
 g_flag_procviminfolines = get_boolean_flag("v"); \
 g_regex_ws = "[ \\t]"; \
 g_regex_ws_zeroormore = g_regex_ws "*"; \
 g_regex_ws_oneormore = g_regex_ws g_regex_ws "*"; \
 \
 if ( ( length( g_comment_eolstr ) == 0 ) && g_flag_isstandardscript ) { \
  g_comment_eolstr = "#"; \
 } \
 if ( length( g_comment_eolstr ) > 0 ) { \
  g_comment_regex = "^" g_regex_ws_zeroormore g_comment_eolstr; \
 } \
 if ( g_flag_procviminfolines ) { \
  if ( length( g_comment_regex ) > 0 ) { \
   g_viminfolines_regex = \
    g_comment_regex \
    g_regex_ws_oneormore \
    "(vi|vim|ex):" \
    g_regex_ws_zeroormore \
    ; \
  } \
 } \
 if ( g_flag_addfileproccomments ) { \
  g_fileproc_comment_data_pref_start = g_comment_eolstr " "; \
  g_fileproc_comment_data_pref_end = g_comment_eolstr " }}} "; \
  g_fileproc_comment_data_suff_start = " {{{"; \
  g_fileproc_comment_data_suff_end = ""; \
 } \
} \
\
{ \
 r_flag_removeblanklines = g_flag_removeblanklines; \
 r_filename = FILENAME; \
} \
( g_flag_addfileproccomments ) { \
 if ( r_filename != g_current_filename ) { \
  proc_on_end_of_file(); \
 } \
} \
( g_flag_procviminfolines && ( match( $$0, g_viminfolines_regex ) > 0 ) ) { \
 g_viminfo_lastline = $$0; \
} \
( g_flag_removeallcomments && ( match( $$0, g_comment_regex ) > 0 ) ) { \
 sub( g_comment_regex ".*$$", "" ); \
 r_flag_removeblanklines = 1; \
} \
( r_flag_removeblanklines && /^[ \\t]*$$/ ) { \
 next; \
} \
( g_flag_addfileproccomments ) { \
 if ( r_filename != g_current_filename ) { \
  r_current_filename_comments_leadingblanks = $$0; \
  sub( "[^ \\t].*$$", "", r_current_filename_comments_leadingblanks ); \
  g_current_filename = r_filename; \
  print \
   r_current_filename_comments_leadingblanks \
   g_fileproc_comment_data_pref_start \
   g_current_filename \
   g_fileproc_comment_data_suff_start \
   ; \
 } \
} \
\
{ print } \
END { \
 proc_on_end_of_file(); \
 if ( g_flag_procviminfolines && ( length( g_viminfo_lastline ) > 0 ) ) { \
  print g_viminfo_lastline; \
 } \
} \
 ' $(1) > '$(2)'
endef

# args:
# $(1): *every* variable prefix (example: "FILE_ENV_")
#
# MAYBE: make every parameter after the one(s) above optional, and eventually
# remove them (and only use variables).
#
# $(2): common file prefix (example: ".env")
#       NOTE: '+' uses a $(1)-prefixed default, '' means unspecified/unset.
#       '+': $(1)SRC_COMP_BASE
# $(3): [optional] if non-empty, it creates a target that depends on the
#       '$(1)SRC_ALL' variable.
#       NOTE: '+' uses a $(1)-prefixed default, '' means unspecified/unset.
#       '+': $(1)TARGET (if non-empty), or the calculated value for $(2)
#            otherwise.
# $(4): [optional] additional "hostnames" (for example "fallback default")
#       to be used;
#       NOTE: '+' uses a $(1)-prefixed default, '' means unspecified/unset.
#       '+': $(1)HOSTNAMES
#       #-? '*': TODO: a multi-target default list ('FILE_ALL_HOSTNAMES'?)
#           (not needed, we have 'HOSTNAMES_ALL' to add to every var-specific
#           list)
# $(5): [optional] flags (default: empty)
#       NOTE: '+' uses a $(1)-prefixed default, '' means unspecified/unset.
#       '+': $(1)CALCFILESCOMMONANDHOST_OPTIONS
#       '*': CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT
#       * common flags:
#        'h': require a hostname-based name to be present (this would include
#             additional hostnames given in $(4)).
#        'H': like 'h', but only enforce when there are no known sub-make
#             invocations to be made.
#       * when a target is created:
#        'A': get rid of every comment (not just the "recognised" ones);
#        'B': get rid of blank lines;
#        'i': add information (for each include file) comments;
#        's': comments use the standard script "whitespace hash to end of line"
#             script convention;
#        'v': keep 'vi'/'vim' sensitive content (usually inside comments);
#         * 'vi'/'vim' "modelines";
#        TODO: implement some or all of the following
#        'C': get rid of recognised comments:
#         * 'vi'/'vim' "modelines";
# TODO: implement the following
# $(6): [optional] "end-of-line" comment prefixes;
#       NOTE: if the 's' flag was specified, then this is not needed.
#
# additional input(s):
#  * $(1)SUBMAKE_STD
#   * for each of these files, $(MAKE) will be run with some of the same
#     variable values used in *this* makefile (to be defined);
#     NOTE: this uses '$(1)SUBMAKE_PHONYTARGETNAME' (see below);
#  * $(1)SUBMAKE_PHONYTARGETNAME
#  * $(1)COMMONANDHOST_PARTS_DIRS:
#    [optional] list of directories to consider for additional parts to be
#    searched in.
#  * ALL_COMMONANDHOST_PARTS_DIRS
# output:
# * defines the follwing variables:
#    $(1)SRC_ALL
#     $(1)SRC_COMMON
#     $(1)SRC_THISHOST
#     $(1)SRC_THISTAGID
# * if $(3) is non-empty, then:
#  * a target file based on the value of $(3) is added to 'TARGETS_FILES';
#  * a rule based on $(1)SRC_ALL that 'cat's every file, catering for files not
#    ending with a newline.  This means that it is possible for this rule to
#    create a file with empty lines between source files.
define EXPR_CALCFILESCOMMONANDHOST
t_vn_pref    := $$(strip $(1))
t_fn_pref    := $$(strip $(2))
t_targetfile := $$(strip $(3))
t_hostall    := $$(strip $(4))
t_opts       := $$(strip $(5))
t_tgtcommenteolstr := $$(strip $(6))
# possibly to be taken from parameters, too
t_tagidsall := +

ifeq ($$(t_fn_pref),+)
 t_fn_pref := $$(strip $$($$(t_vn_pref)SRC_COMP_BASE))
endif

ifeq ($$(t_targetfile),+)
 t_targetfile := $$(strip $$($$(t_vn_pref)TARGET))
 ifeq ($$(t_targetfile),)
  t_targetfile := $$(t_fn_pref)
 endif
endif

ifeq ($$(t_hostall),+)
 t_hostall := $$(strip $$($$(t_vn_pref)HOSTNAMES))
endif
t_hostall := $$(strip $$(t_hostall) $$(HOSTNAMES_ALL))
t_hostall := $$(call FUNC_WORDLIST_REMOVEDUPLICATES,$$(t_hostall))

ifeq ($$(t_tagidsall),+)
 t_tagidsall := $$(strip $$($$(t_vn_pref)ALL_TAGIDS))
endif
#+ prev: v1: t_tagidsall := $$(strip $$(t_tagidsall) $$(ALL_TAGIDS_ALL))
t_tagidsall := $$(strip \
  )$$(call FUNC_TAGMAPPING_EXPANDTAGIDS,$$(strip \
   )$$(ALL_TAGIDS_TAGMAPPING),$$(strip \
   )$$(t_tagidsall) $$(ALL_TAGIDS_ALL)$$(strip \
  ))$$(strip \
 )

ifeq ($$(t_opts),+)
 t_opts := $$(strip $$($$(t_vn_pref)CALCFILESCOMMONANDHOST_OPTIONS))
endif
ifeq ($$(t_opts),*)
 t_opts := $$(strip $$(CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT))
endif

ifneq ($$(VERBOSE),)
 $$(info INFO: EXPR_CALCFILESCOMMONANDHOST(): started)
 $$(info INFO: variables prefix: '$$(t_vn_pref)')
 $$(info INFO: src filenames prefix: '$$(t_fn_pref)')
 $$(info INFO: optional target filename: '$$(t_targetfile)')
 $$(info INFO: additional hostnames to consider: '$$(t_hostall)')
 $$(info INFO: (smart cat/own) options flags (pre/orig): '$$(t_opts)')
 $$(info INFO: active tagids for this variable prefix: '$$(t_tagidsall)')
endif

$$(t_vn_pref)SRC_COMMON := $(strip \
 $$(call FUNC_GETALLFILESFORFIRSTMATCHINGBASENAME,$$(strip $$(strip \
   )$$(t_fn_pref)-common $$(strip \
  )),$$(strip $$(strip \
   )$$($$(t_vn_pref)COMMONANDHOST_PARTS_DIRS) $$(strip \
  ))$$(strip \
 ))$$(strip \
))
$$(t_vn_pref)SRC_THISHOST := $$(strip \
 $$(call FUNC_GETALLFILESFORFIRSTMATCHINGBASENAME,$$(strip $$(strip \
   )$$(patsubst %,$$(t_fn_pref)-host-%,$$(t_hostall)) $$(strip \
  )),$$(strip $$(strip \
   )$$($$(t_vn_pref)COMMONANDHOST_PARTS_DIRS) $$(strip \
  ))$$(strip \
 ))$$(strip \
))
$$(t_vn_pref)SRC_THISTAGID := $$(strip \
 $$(call FUNC_GETALLFILESFORALLMATCHINGBASENAMES,$$(strip $$(strip \
   )$$(patsubst %,$$(t_fn_pref)-tagid-%,$$(t_tagidsall)) $$(strip \
  )),$$(strip $$(strip \
   )$$($$(t_vn_pref)COMMONANDHOST_PARTS_DIRS) $$(strip \
  ))$$(strip \
 ))$$(strip \
))

ifneq ($$(VERBOSE),)
 $$(info INFO: '$$(t_fn_pref)'-prefixed file(s) (common): '$$($$(t_vn_pref)SRC_COMMON)')
 $$(info INFO: '$$(t_fn_pref)'-prefixed file(s) for this host: '$$($$(t_vn_pref)SRC_THISHOST)')
 $$(info INFO: '$$(t_fn_pref)'-prefixed file(s) for the current tagid: '$$($$(t_vn_pref)SRC_THISTAGID)')
endif

ifneq ($$(VERBOSE),)
 $$(info INFO: $$(t_vn_pref)SUBMAKE_STD: '$$($$(t_vn_pref)SUBMAKE_STD)')
endif
#
# TODO: check whether these variables are expected to be defined, or if they
# happen to be defined later (through other macros).
# if there are no sub-make invocations to be had, we will enforce the 'H' flag
# as 'h' normally is.
ifeq ($$(strip $$($$(t_vn_pref)SUBMAKE_STD)),)
 t_opts := $$(subst H,h,$$(t_opts))
endif

ifneq ($$(VERBOSE),)
 $$(info INFO: (smart cat/own) options flags (post): '$$(t_opts)')
endif

# (optionally) complain if there is no 'env' file for this host
ifneq ($$(findstring h,$$(t_opts)),)
 ifeq ($$(strip $$($$(t_vn_pref)SRC_THISHOST)$$($$(t_vn_pref)SRC_THISTAGID)),)
  $$(error could not find neither a '$$(t_fn_pref)-host-*' file for this host for any of these hostnames: '$$(t_hostall)', nor a '$$(t_fn_pref)-tagid-*' file for any of these tagids: '$$(t_tagidsall)'.)
 endif
endif

$$(t_vn_pref)SRC_ALL := $$(strip \
 $$(call FUNC_WORDS_SORTBYPARTNUMBER,$$(strip \
  $$($$(t_vn_pref)SRC_COMMON) \
  $$($$(t_vn_pref)SRC_THISHOST) \
  $$($$(t_vn_pref)SRC_THISTAGID) \
  $$($$(t_vn_pref)SUBMAKE_STD) \
 ),$$(strip \
 .part- .part_ \
 ),99,b) \
 )

ifneq ($$(strip $$($$(t_vn_pref)SUBMAKE_STD)),)
 ALL_SUBMAKE_STD_VARNAME_PREFS += $$(t_vn_pref)
endif

ifneq ($$(t_targetfile),)
 TARGETS_FILES += $$(t_targetfile)
 ifneq ($$(VERBOSE),)
  $$(info INFO: current target file: '$$(t_targetfile)')
  $$(info INFO: TARGETS_FILES: '$$(TARGETS_FILES)')
 endif

 # TODO: check that t_targetfile could actually be in $$(t_vn_pref)SRC_ALL
 #  (and therefore making sure that this next assignment is necessary)
 $$(t_vn_pref)SRC_ALL := $$(filter-out $$(strip \
  )$$(t_targetfile),$$(strip \
  )$$($$(t_vn_pref)SRC_ALL)$$(strip \
  ))

 # NOTE: we can't use variables whose name depend on re-assigned variables,
 # like $(t_vn_pref), we have to use $(1), which is expanded at '$(call )'
 # time, as recipes are evaluated at the point where they're executed, not when
 # they're defined.
 # args: $$(1): $$+ in the rule
 $(1)FUNC_RULE_TARGET_INTERNAL_GET_SRC_FILES = $$(strip \
  )$$(filter-out $$(SRC_FILES_COMMON),$$(1))$$(strip \
  )

 $(1)RULE_TARGET_INTERNAL_VAL_OPTS := $$(strip $$(t_opts))

 $(1)RULE_TARGET_INTERNAL_VAL_TGTCOMMEOLSTR := $$(strip $$(t_tgtcommenteolstr))

 # args:
 # $$(1): $$@ in the rule
 # $$(2): $$+ in the rule
 #+ prev: v1: $(1)FUNC_RULE_TARGET_INTERNAL_CMD = $$(strip \
 #+ prev: v1:  $$(info creating file '$$(1)' from $$(strip \
 #+ prev: v1:   )$$(call $(1)FUNC_RULE_TARGET_INTERNAL_GET_SRC_FILES,$$(2)) . . .$$(strip \
 #+ prev: v1:  ))$$(strip \
 #+ prev: v1:   )$$(call CMD_CATALLFILES_SMART,$$(strip \
 #+ prev: v1:    )$$(call $(1)FUNC_RULE_TARGET_INTERNAL_GET_SRC_FILES,$$(2)),$$(strip \
 #+ prev: v1:    )$$(1),$$(strip \
 #+ prev: v1:    )$$($(1)RULE_TARGET_INTERNAL_VAL_OPTS),$$(strip \
 #+ prev: v1:    )$$($(1)RULE_TARGET_INTERNAL_VAL_TGTCOMMEOLSTR)$$(strip \
 #+ prev: v1:   )) $$(strip \
 #+ prev: v1:   )|| { rm -vf $$(1) ; false ; }$$(strip \
 #+ prev: v1:   )$$(strip \
 #+ prev: v1:  ))
 $(1)FUNC_RULE_TARGET_INTERNAL_CMD = $$(strip \
  )echo "creating file '$$(1)' from $$(strip $$(strip \
    )$$(call $(1)FUNC_RULE_TARGET_INTERNAL_GET_SRC_FILES,$$(2)) . . .$$(strip \
   ))"$$(strip \
  ) && $$(strip \
  )$$(strip \
  )$$(call CMD_CATALLFILES_SMART,$$(strip \
   )$$(call $(1)FUNC_RULE_TARGET_INTERNAL_GET_SRC_FILES,$$(2)),$$(strip \
   )$$(1),$$(strip \
   )$$($(1)RULE_TARGET_INTERNAL_VAL_OPTS),$$(strip \
   )$$($(1)RULE_TARGET_INTERNAL_VAL_TGTCOMMEOLSTR)$$(strip \
  ))$$(strip \
  ) || { rm -vf $$(1) ; false ; }$$(strip \
   )$$(strip \
  )

 $$(t_targetfile): $$(strip $$($$(t_vn_pref)SRC_ALL)) $$(SRC_FILES_COMMON)
	@$$(call $(1)FUNC_RULE_TARGET_INTERNAL_CMD,$$@,$$+)
endif

ifneq ($$(VERBOSE),)
 $$(info INFO: $$(t_vn_pref)SRC_ALL: '$$($$(t_vn_pref)SRC_ALL)')
endif

.PRECIOUS: $$($$(t_vn_pref)SRC_ALL)

undefine t_vn_pref
undefine t_fn_pref
undefine t_targetfile
#- v2: undefine t_srcfiles_data
undefine t_hostall
undefine t_opts
undefine t_tgtcommenteolstr
endef

ALL_COMMONANDHOST_PARTS_DIRS_BASEDIR ?=
ifeq ($(origin ALL_COMMONANDHOST_PARTS_DIRS_DEFAULT),undefined)
 # examples for leaf dirnames:
 #  dockercomposemake-parts.d-common_parent_01
 ALL_COMMONANDHOST_PARTS_DIRS_DEFAULT := $(strip $(strip \
  )$(wildcard $(strip \
   )$(addprefix $(strip $(strip \
     )$(call FUNC_GETDIRPREFANDSEPARATOR,$(strip \
      $(ALL_COMMONANDHOST_PARTS_DIRS_BASEDIR) \
     ))$(strip \
    )),$(strip \
    )dockercomposemake-parts.d-* $(strip \
   ))$(strip \
  ))$(strip \
 ))
endif

# these might be useful for 'include'd makefiles, so we place these before the
# 'include' statement/command.
FILE_ENV_TARGET_DEFAULT ?= .env
FILE_DOCKERCOMPOSEYML_TARGET_DEFAULT ?= docker-compose.yml

# SUBMAKE_STD-related utility FUNC_*, EXPR_*, EVAL_* symbols {{{

# args:
# $(1): (sub/)dir for the file to be generated in;
#?  (
#? )$(subst __,_,$(strip $(strip \
#?  ))
define FUNC_MAK_GET_PARTID_FROM_DIR
$(strip $(strip \
 )$(subst /,_,$(strip $(strip \
   )$(1)$(strip \
  ))$(strip \
 ))$(strip \
))
endef

# args:
# $(1): (sub/)dir for the file to be generated in;
# $(2): base for the filename to be generated (before the '-common.part-');
# $(3): partid (usually a 2-digit number);
define FUNC_MAK_MAKE_SUBMAKESTD_ENTRY_FROM_DIR_AND_BASE
$(strip $(strip \
 )$(call FUNC_GETDIRPREFANDSEPARATOR,$(1))$(strip \
 )$(strip $(2))-stdsubmake.part-$(strip $(3))_$(strip \
 )$(call FUNC_MAK_GET_PARTID_FROM_DIR,$(1))$(strip \
))
endef

# args:
# $(1): variable prefix (example: 'FILE_DOCKERCOMPOSEYML_');
# $(2): (sub/)dir for the file to be generated in;
# $(3): partid (usually a 2-digit number);
define EVAL_MAK_ADD_SUBMAKESTD_ENTRY_FORVARPREF
$(eval $(strip \
 )$(1)SUBMAKE_STD += $(strip \
  )$$(call FUNC_MAK_MAKE_SUBMAKESTD_ENTRY_FROM_DIR_AND_BASE,$(strip \
   )$$(strip $(2)),$(strip \
   )$$($(1)TARGET_DEFAULT),$(strip \
   )$$(strip $(3))$(strip \
  ))$(strip \
))
endef

# args:
# $(1): (sub/)dir for the file to be generated in;
# $(2): partid (usually a 2-digit number);
define EVAL_MAK_ADD_SUBMAKESTD_ENTRY_DOCKERCOMPOSEYML
$(call EVAL_MAK_ADD_SUBMAKESTD_ENTRY_FORVARPREF,$(strip \
 )FILE_DOCKERCOMPOSEYML_,$(strip \
 )$(1),$(strip \
 )$(2)$(strip \
))
endef

# }}}

ifeq (new,old)

# by default, our makefile inclusion process uses the automatically detected
# "common parts" directories, if any.
MAK_INCLUDEFILES_COMMONANDHOST_PARTS_DIRS ?= $(strip \
 $(ALL_COMMONANDHOST_PARTS_DIRS_DEFAULT) \
)

# include "make"-configuration files
#  IDEA: move the macros to expand the wildcard based on the hostname, etc.,
#  and provide a nice "null" fallback/default file, so no overrides are made.
$(eval $(call EXPR_CALCFILESCOMMONANDHOST,$(strip \
 )MAK_INCLUDEFILES_,$(strip \
 )$(notdir $(MAK_MAIN_MAKEFILE)),$(strip \
 ))$(strip \
))

# every target also depends on the makefiles being included.
SRC_FILES_COMMON += $(MAK_INCLUDEFILES_SRC_ALL)

ifneq ($(VERBOSE),)
 $(info INFO: current makefile: MAK_THIS_MAKEFILE: '$(MAK_THIS_MAKEFILE)')
 $(info INFO: main makefile: MAK_MAIN_MAKEFILE: '$(MAK_MAIN_MAKEFILE)')
 $(info INFO: MAK_THIS_ISDOCKERCOMPOSEMAKE_SUBMAKE: '$(MAK_THIS_ISDOCKERCOMPOSEMAKE_SUBMAKE)')
 $(info INFO: main makefile for sub-make: MAK_MAIN_MAKEFILE_FORSUBMAKE: '$(MAK_MAIN_MAKEFILE_FORSUBMAKE)')
 $(info INFO: MAK_INCLUDEFILES_SRC_ALL (before including other makefiles): '$(MAK_INCLUDEFILES_SRC_ALL)')
endif

include $(MAK_INCLUDEFILES_SRC_ALL)

else

# args:
# $(1): variables prefix (passed to 'EXPR_CALCFILESCOMMONANDHOST');
define EXPR_INCLUDEFILES_STEP
t_incfiles_vn_pref := $$(strip $(1))

# by default, our makefile inclusion process uses the automatically detected
# "common parts" directories, if any.
$$(t_incfiles_vn_pref)COMMONANDHOST_PARTS_DIRS ?= $$(strip \
 $$(ALL_COMMONANDHOST_PARTS_DIRS_DEFAULT) \
)

# include "make"-configuration files
#  IDEA: move the macros to expand the wildcard based on the hostname, etc.,
#  and provide a nice "null" fallback/default file, so no overrides are made.
$$(eval $$(strip \
 )$$(call EXPR_CALCFILESCOMMONANDHOST,$$(strip \
  )$$(t_incfiles_vn_pref),$$(strip \
  )$$(notdir $$(MAK_MAIN_MAKEFILE)),$$(strip \
 ))$$(strip \
))

# include files that have not been processed so far.
# (in other words, remove the files that have been included already).
$$(t_incfiles_vn_pref)SRC_ALL := $$(strip $$(strip \
 )$$(filter-out $$(MAKEFILE_LIST),$$($$(t_incfiles_vn_pref)SRC_ALL))$$(strip \
))

# every target also depends on the makefiles being included.
SRC_FILES_COMMON += $$($$(t_incfiles_vn_pref)SRC_ALL)

ifneq ($$(VERBOSE),)
 $$(info INFO: makefile inclusion: start. varpref='$$(t_incfiles_vn_pref)')
 $$(info INFO:  current makefile: MAK_THIS_MAKEFILE: '$$(MAK_THIS_MAKEFILE)')
 $$(info INFO:  main makefile: MAK_MAIN_MAKEFILE: '$$(MAK_MAIN_MAKEFILE)')
 $$(info INFO:  MAK_THIS_ISDOCKERCOMPOSEMAKE_SUBMAKE: '$$(MAK_THIS_ISDOCKERCOMPOSEMAKE_SUBMAKE)')
 $$(info INFO:  main makefile for sub-make: MAK_MAIN_MAKEFILE_FORSUBMAKE: '$$(MAK_MAIN_MAKEFILE_FORSUBMAKE)')
 $$(info INFO:  ALL_TAGIDS_ALL: '$$(ALL_TAGIDS_ALL)')
 $$(info INFO:  $$(t_incfiles_vn_pref)SRC_ALL (before including other makefiles): '$$($$(t_incfiles_vn_pref)SRC_ALL)')
endif

include $$($$(t_incfiles_vn_pref)SRC_ALL)

undefine  t_incfiles_vn_pref
endef

# include files twice, to give makefiles a chance to define a "tagid", for
# example, and then have files corresponding to the "tagid" being included
# (and later processed for other targets, too).
$(foreach vnpref,$(strip $(strip \
  )MAK_INCLUDEFILES_PRE_ $(strip \
  )MAK_INCLUDEFILES_POST_ $(strip \
 )),$(strip \
 )$(eval $(strip \
  )$(call EXPR_INCLUDEFILES_STEP,$(vnpref))$(strip \
 ))$(strip \
))
# TESTING: $(error finished early (testing))

endif # old/new implementation

SRC_FILES_COMMON := $(call FUNC_WORDLIST_REMOVEDUPLICATES,$(strip \
  )$(SRC_FILES_COMMON)$(strip \
 ))

ALL_SUBMAKE_STD_FORCEBUILD ?= 1
ifneq ($(filter 1,$(ALL_SUBMAKE_STD_FORCEBUILD)),)
 ALL_SUBMAKE_STD_EXTRADEPS += submake_std_builddep
endif

#+ prev: ifneq ($(VERBOSE),)
#+ prev:  $(info INFO: makefile inclusion: end)
#+ prev:  $(info INFO:  SRC_FILES_COMMON: '$(SRC_FILES_COMMON)')
#+ prev: endif

ifneq ($(VERBOSE),)
 $(info INFO: finished including makefiles. MAKEFILE_LIST: '$(MAKEFILE_LIST)')
 $(info INFO: SRC_FILES_COMMON: '$(SRC_FILES_COMMON)')
endif

ifneq ($(VERBOSE),)
 $(info INFO: hostnames from the OS: $(HOSTNAMES_ALL_ORIG))
endif

ifneq ($(VERBOSE),)
 $(foreach tokenid,SPACE NEWLINE,$(info INFO: $(tokenid)='$($(tokenid))'))
endif

ifneq ($(VERBOSE),)
 $(info TOKEN_SPACE: '$(TOKEN_SPACE)')
 $(info TOKEN_NEWLINE: '$(TOKEN_NEWLINE)')
endif

CMD_DOCKER_COMPOSE ?= docker-compose
CMD_SUDO ?= sudo
CMD_AWK ?= awk

ifeq ($(origin CMD_SUBMAKE_START),undefined)
 CMD_SUBMAKE_START := $(strip \
  )$(if $(VERBOSE),,@)$(strip \
  )+$(strip \
  )
endif

ifeq ($(origin ALL_COMMONANDHOST_PARTS_DIRS),undefined)
 ALL_COMMONANDHOST_PARTS_DIRS := $(ALL_COMMONANDHOST_PARTS_DIRS_DEFAULT)
endif
ifneq ($(VERBOSE),)
 $(info ALL_COMMONANDHOST_PARTS_DIRS_DEFAULT: '$(ALL_COMMONANDHOST_PARTS_DIRS_DEFAULT)')
 $(info ALL_COMMONANDHOST_PARTS_DIRS: '$(ALL_COMMONANDHOST_PARTS_DIRS)')
endif

# done: allow *all* tagids to be processed (unlike hosts), so:
#  . refactor current macro(s) to have a flavour where *all* matches for *all*
#    words are returned;
#
#    . ... and refactor the existing FUNC_GETALLFILESFORFIRSTMATCHINGBASENAME to use
#      those new macros.

# TODO: where "sub-make" files are being specified (non-empty list), we probably don't need to force the 'h' flag.
#  IDEA: either don't specify it by default, or:
#  IDEA: ignore it when the "sub-make" list is non-empty, or:
#  TODO: have a new flag that has that conditional enforcement;

#+ v1: CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT ?= h
#+ v2: CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT ?= hAB
#+ v3: CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT := hABv
ifeq ($(origin CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT),undefined)
 CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT := ABv
 ifeq ($(MAK_THIS_ISDOCKERCOMPOSEMAKE_SUBMAKE),)
  CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT += H
 endif
 ifneq ($(VERBOSE),)
  CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT += i
 endif
 # NOTE: the value for $(CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT)
 # might have embedded spaces (the '+=' adds one, for example), but that should
 # still work as expected (specified flag values are checked for presence,
 # ignoring other characters (such as spaces), duplicates, etc.)
endif

ifeq ($(origin SUDO_NEEDED),undefined)
 UID := $(shell id -u)
 GROUPNAMES := $(shell id -G -n ; id -g -n)
 SUDO_NEEDED := $(if $(strip \
  )$(filter docker,$(GROUPNAMES))$(strip \
  )$(filter 0,$(UID))$(strip \
  ),0,1)
endif

# TODO: pass the variable prefix and '+' as the value for the other fields to
# mean "calculate the default from the other parameter(s)", so on most calls we
# will either specify a blank (as is the case when calculating the files to
# "include"), or + (as for 'env' and 'dockercomposeyml').
#
# prev: v1: FILE_ENV_TARGET = .env
FILE_ENV_SRC_COMP_BASE ?= $(FILE_ENV_TARGET_DEFAULT)
FILE_ENV_TARGET ?= $(FILE_ENV_TARGET_DEFAULT)
FILE_ENV_SUBMAKE_PHONYTARGETNAME := env
FILE_ENV_CALCFILESCOMMONANDHOST_OPTIONS ?= $(strip $(strip \
 )$(CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT)$(strip \
 )s$(strip \
))
#
# prev: v1: env : $(FILE_ENV_TARGET)
$(FILE_ENV_SUBMAKE_PHONYTARGETNAME) : $(FILE_ENV_TARGET)
#+ TARGETS_FILES += $(FILE_ENV_TARGET)

# prev: v1: FILE_DOCKERCOMPOSEYML_TARGET = docker-compose.yml
FILE_DOCKERCOMPOSEYML_SRC_COMP_BASE ?= $(FILE_DOCKERCOMPOSEYML_TARGET_DEFAULT)
FILE_DOCKERCOMPOSEYML_TARGET ?= $(FILE_DOCKERCOMPOSEYML_TARGET_DEFAULT)
FILE_DOCKERCOMPOSEYML_SUBMAKE_PHONYTARGETNAME := dockercomposeyml
FILE_DOCKERCOMPOSEYML_CALCFILESCOMMONANDHOST_OPTIONS ?= $(strip $(strip \
 )$(CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT)$(strip \
 )s$(strip \
))
#
# prev: v1: dockercomposeyml : $(FILE_DOCKERCOMPOSEYML_TARGET)
$(FILE_DOCKERCOMPOSEYML_SUBMAKE_PHONYTARGETNAME) : \
 $(FILE_DOCKERCOMPOSEYML_TARGET)
#+ TARGETS_FILES += $(FILE_DOCKERCOMPOSEYML_TARGET)

#+ prev: files : env dockercomposeyml
files : \
 $(FILE_ENV_SUBMAKE_PHONYTARGETNAME) \
 $(FILE_DOCKERCOMPOSEYML_SUBMAKE_PHONYTARGETNAME) \
 # end

# (target) aliases
up : compose-up
stop : compose-stop
docker-clean : compose-rm

clean ::
	@files='$(wildcard $(TARGETS_FILES))' && \
		{ [ -z "$$files" ] || rm -fv $$files ; }

#+ FILE_ENV_SRC_COMMON_PRE := $(firstword $(wildcard .env-common-pre))
#+ FILE_ENV_SRC_COMMON_POST := $(firstword $(wildcard .env-common-post))
#+ FILE_ENV_SRC_THISHOST := $(strip \
#+  $(firstword $(wildcard $(patsubst %,.env-host-%,$(HOSTNAMES_ALL)))))
#+ ifneq ($(VERBOSE),)
#+  $(info INFO: .env file for this host: '$(FILE_ENV_SRC_THISHOST)')
#+ endif
#+ # complain if there is no 'env' file for this host
#+ ifeq ($(FILE_ENV_SRC_THISHOST),)
#+  $(error could not find an '.env-host-*' file for this host for any of these hostnames: $(HOSTNAMES_ALL))
#+ endif
#+ 
#+ $(FILE_ENV_TARGET) : \
#+  $(FILE_ENV_SRC_COMMON_PRE) \
#+  $(FILE_ENV_SRC_THISHOST) \
#+  $(FILE_ENV_SRC_COMMON_POST) \
#+  # end
#+ 	@{ $(foreach f,$+,cat $(f) && echo &&) : ; } > $@
#? $(eval $(call EXPR_CALCFILESCOMMONANDHOST,$(strip \
#?  )FILE_ENV_SRC_,.env,+))
# testing: $(info expanded: $(call EXPR_CALCFILESCOMMONANDHOST,$(strip \
# testing:  )FILE_ENV_SRC_,.env,+))
#+ $(eval $(call EXPR_CALCFILESCOMMONANDHOST,$(strip \
#+  )FILE_ENV_SRC_,.env,+))
# prev: v1: $(eval $(call EXPR_CALCFILESCOMMONANDHOST,$(strip \
# prev: v1:  )FILE_ENV_SRC_,$(FILE_ENV_TARGET),+,,$(strip \
# prev: v1:  )$(CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT)s))
# prev: v2: $(eval $(call EXPR_CALCFILESCOMMONANDHOST,$(strip \
# prev: v2:  )FILE_ENV_SRC_,$(strip \
# prev: v2:  )$(FILE_ENV_SRC_COMP_BASE),$(strip \
# prev: v2:  )$(FILE_ENV_TARGET),$(strip \
# prev: v2:  ),$(strip \
# prev: v2:  )$(CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT)s))
$(eval $(call EXPR_CALCFILESCOMMONANDHOST,$(strip \
 )FILE_ENV_,$(strip \
 )+,$(strip \
 )+,$(strip \
 )+,$(strip \
 )+,$(strip \
 ))$(strip \
))

#+ $(eval $(call EXPR_CALCFILESCOMMONANDHOST,$(strip \
#+  )FILE_DOCKERCOMPOSEYML_SRC_,docker-compose.yml,+))
# prev: v2: $(eval $(call EXPR_CALCFILESCOMMONANDHOST,$(strip \
# prev: v2:  )FILE_DOCKERCOMPOSEYML_SRC_,$(FILE_DOCKERCOMPOSEYML_TARGET),+,,$(strip \
# prev: v2:  )$(CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT)s))
# prev: v3: $(eval $(call EXPR_CALCFILESCOMMONANDHOST,$(strip \
# prev: v3:  )FILE_DOCKERCOMPOSEYML_SRC_,$(strip \
# prev: v3:  )$(FILE_DOCKERCOMPOSEYML_SRC_COMP_BASE),$(strip \
# prev: v3:  )$(FILE_DOCKERCOMPOSEYML_TARGET),$(strip \
# prev: v3:  ),$(strip \
# prev: v3:  )$(CALCFILESCOMMONANDHOST_OPTIONS_TARGETFILES_DEFAULT)s))
$(eval $(call EXPR_CALCFILESCOMMONANDHOST,$(strip \
 )FILE_DOCKERCOMPOSEYML_,$(strip \
 )+,$(strip \
 )+,$(strip \
 )+,$(strip \
 )+,$(strip \
 ))$(strip \
))

# testing messages
ifneq ($(VERBOSE),)
 $(info INFO: (near end of file) TARGETS_FILES: '$(TARGETS_FILES)')
endif

TARGETS_ALL = $(TARGETS_PHONY) $(TARGETS_FILES)

#? untested: ifneq ($(strip $(ALL_SUBMAKE_STD)),)
#? untested:  # call $(MAKE) recursively for $(TARGETS_PHONY_RECURSIVE) for each directory
#? untested:  # that generates targets for any of our main "phony" file-related targets.
#? untested:  $(foreach fdd,$(strip \
#? untested:   $(sort $(dir $(strip \
#? untested:    $(ALL_SUBMAKE_STD) \
#? untested:   ))),
#? untested:   ),$$(strip \
#? untested:   )$(foreach tgt,$(TARGETS_PHONY_RECURSIVE),$(strip \
#? untested:    )$(eval $(tgt) ::$(NEWLINE)$(strip \
#? untested:     )$(TAB)@$$(MAKE) -C $(fdd) $$@$(NEWLINE)$(strip \
#? untested:    ))$(strip \
#? untested:   ))$(strip \
#? untested:  ))
#? untested: endif

ifneq ($(strip $(ALL_SUBMAKE_STD_VARNAME_PREFS)),)

# args:
# $(1): dir (key);
# $(2): make opts (can contain spaces, which will be '$(subst )'-ed).
#? prev: v1: define FUNC_SUBMAKE_STD_DATABYDIR_MAKEENTRY_MAKEOPTS
#? prev: v1: $(strip \
#? prev: v1: )$(call FUNC_GET_SUBMAKE_STD_MAKEENTRYFIELDSTRFROMVALUE,$(1))$(strip \
#? prev: v1: )$(TOKEN_FIELDSEP)$(strip \
#? prev: v1: )$(TOKEN_NULL)$(strip \
#? prev: v1: )$(TOKEN_FIELDSEP)$(strip \
#? prev: v1: )$(TOKEN_NULL)$(strip \
#? prev: v1: )$(TOKEN_FIELDSEP)$(strip \
#? prev: v1: )$(call FUNC_GET_SUBMAKE_STD_MAKEENTRYFIELDSTRFROMVALUE,$(2))$(strip \
#? prev: v1: )
#? prev: v1: endef
define FUNC_SUBMAKE_STD_DATABYDIR_MAKEENTRY_MAKEOPTS
$(call FUNC_SUBMAKE_STD_DATABYDIR_MAKEENTRY_FIELDNUM,$(strip \
 )$(1),$(strip \
 )$(ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_MAKEOPTS),$(strip \
 )$(2)$(strip \
))
endef

# args:
# $(1): field value (can contain spaces, which will be '$(subst )'-ed).
define FUNC_GET_SUBMAKE_STD_MAKEENTRYFIELDSTRFROMVALUE
$(subst $(SPACE),$(TOKEN_SPACE),$(1))
endef

# args:
# $(1) field (usually $(TOKEN_FIELDSEP)-separated)
#+ prev: v1: $(subst $(TOKEN_SPACE),$(SPACE),$(1))
define FUNC_GET_SUBMAKE_STD_VALUEFROMENTRYFIELDSTR
$(strip \
)$(subst $(TOKEN_NULL),,$(strip \
)$(subst $(TOKEN_SPACE),$(SPACE),$(strip \
)$(1)$(strip \
))$(strip \
))$(strip \
)
endef

# args:
# $(1): entry (usually from $(ALL_SUBMAKE_STD_DATABYDIR_MULTIPLE));
# $(2): field number;
define FUNC_GET_SUBMAKE_STD_STRFROMENTRYFIELDNUM
$(word $(2),$(subst $(TOKEN_FIELDSEP),$(SPACE),$(1)))
endef

# args:
# $(1): entry (usually from $(ALL_SUBMAKE_STD_DATABYDIR_MULTIPLE));
# $(2): field number;
define FUNC_GET_SUBMAKE_STD_VALUEFROMENTRYFIELDNUM
$(call FUNC_GET_SUBMAKE_STD_VALUEFROMENTRYFIELDSTR,$(strip \
 )$(call FUNC_GET_SUBMAKE_STD_STRFROMENTRYFIELDNUM,$(1),$(2))$(strip \
))
endef

# args: $(1) dir
define FUNC_GET_SUBMAKE_STD_ENTRIES_FORDIR
$(filter $(strip $(1))$(TOKEN_FIELDSEP)%,$(ALL_SUBMAKE_STD_DATABYDIR_MULTIPLE))
endef

# args: $(1) dir
define FUNC_GET_SUBMAKE_STD_MAKEOPTS_FORDIR
$(foreach t_entry,$(strip \
  )$(call FUNC_GET_SUBMAKE_STD_ENTRIES_FORDIR,$(1))$(strip \
 ),$(strip \
  $(call FUNC_GET_SUBMAKE_STD_VALUEFROMENTRYFIELDNUM,$(strip \
   )$(t_entry),$(strip \
   )$(ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_MAKEOPTS)$(strip \
  ))$(strip \
 ))$(strip \
))
endef

# args:
# $(1): dir (key);
# $(2): field number (1 would be the key, so avoid);
# $(3): field string (can contain spaces, which will be '$(subst )'-ed).
# TODO: LATER: use a new variable ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_MAX to
# only use that many whitespace-separated words before '$(subst )'-ituting the
# spaces to '$(TOKEN_FIELDSEP)'.
define FUNC_SUBMAKE_STD_DATABYDIR_MAKEENTRY_FIELDNUM
$(strip \
)$(subst $(TOKEN_FIELDSEP)$(TOKEN_FIELDSEP),$(TOKEN_FIELDSEP),$(strip $(strip \
  )$(call FUNC_GET_SUBMAKE_STD_MAKEENTRYFIELDSTRFROMVALUE,$(1))$(strip \
  )$(TOKEN_FIELDSEP)$(strip \
  )$(subst $(SPACE),$(TOKEN_FIELDSEP),$(strip $(strip \
    )$(wordlist 3,$(2),$(subst x,$(TOKEN_NULL)$(SPACE),xxxxxxxxxx))$(strip \
   ))$(strip \
  ))$(strip \
  )$(TOKEN_FIELDSEP)$(strip \
  )$(call FUNC_GET_SUBMAKE_STD_MAKEENTRYFIELDSTRFROMVALUE,$(3))$(strip \
 ))$(strip \
))$(strip \
)
endef

 # construct a list of additional $(MAKE) options to pass to "std" sub-makes,
 # one per "std" make working directory.
 #+ prev: )$(subst $(SPACE),$(TOKEN_SPACE),$(strip \
 #
 ALL_SUBMAKE_STD_DATABYDIR_MULTIPLE := $(strip \
  $(foreach t_vn_pref,$(strip \
    $(ALL_SUBMAKE_STD_VARNAME_PREFS) \
   ),$(strip \
    $(foreach t_fn_any,$(strip \
      $($(t_vn_pref)SUBMAKE_STD) \
     ),$(strip \
      $(dir $(t_fn_any))$(strip \
      )$(TOKEN_FIELDSEP)$(strip \
      )$(t_fn_any)$(strip \
      )$(TOKEN_FIELDSEP)$(strip \
      )$($(t_vn_pref)SUBMAKE_PHONYTARGETNAME)$(strip \
      )$(TOKEN_FIELDSEP)$(strip \
      )$(call FUNC_GET_SUBMAKE_STD_MAKEENTRYFIELDSTRFROMVALUE,$(strip $(strip \
        )$(t_vn_pref)TARGET='$(notdir $(t_fn_any))' $(strip \
       ))$(strip \
      ))$(strip \
     ))$(strip \
    ))$(strip \
   ))$(strip \
  ))$(strip \
 ))
 ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_DIR := 1
 ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_OURTARGETFILE := 2
 ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_PHONYTARGETNAME := 3
 ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_MAKEOPTS := 4

 ALL_SUBMAKE_STD_DIRS := $(strip \
   $(sort $(strip \
     $(foreach t_entry,$(strip \
       )$(ALL_SUBMAKE_STD_DATABYDIR_MULTIPLE)$(strip \
      ),$(strip \
       $(call FUNC_GET_SUBMAKE_STD_VALUEFROMENTRYFIELDNUM,$(strip \
        )$(t_entry),$(strip \
        )$(ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_DIR)$(strip \
       ))$(strip \
      ))$(strip \
     ))$(strip \
    ))$(strip \
   ))$(strip \
  ))

 ifneq ($(VERBOSE),)
  $(info INFO: ALL_SUBMAKE_STD_VARNAME_PREFS: '$(ALL_SUBMAKE_STD_VARNAME_PREFS)')
  $(info INFO: ALL_SUBMAKE_STD_DIRS: '$(ALL_SUBMAKE_STD_DIRS)')
 endif

 # done: add the '-C $(t_dir)' for each directory in $(ALL_SUBMAKE_STD_DIRS),
 # so we don't have to add it ourselves manually to each invocation.
 # done: add the '-f {appropriate_makefile}' for each dir, too
 # ref: # $(1): dir (key);
 # ref: # $(2): make opts (can contain spaces, which will be '$(subst )'-ed).
 # ref: define FUNC_SUBMAKE_STD_DATABYDIR_MAKEENTRY_MAKEOPTS
 ALL_SUBMAKE_STD_DATABYDIR_MULTIPLE += $(strip \
  )$(foreach t_dir,$(ALL_SUBMAKE_STD_DIRS),$(strip $(strip \
    )$(call FUNC_SUBMAKE_STD_DATABYDIR_MAKEENTRY_MAKEOPTS,$(strip \
     )$(t_dir),$(strip \
     )$(strip $(strip \
      )-f $(MAK_MAIN_MAKEFILE_FORSUBMAKE) $(strip \
      )-C $(t_dir) $(strip \
      )MAK_MAIN_MAKEFILE=$(MAK_MAIN_MAKEFILE_FORSUBMAKE) $(strip \
      )MAK_THIS_ISDOCKERCOMPOSEMAKE_SUBMAKE=1 $(strip \
     ))$(strip \
    ))$(strip \
   ))$(strip \
  ))

 ifneq ($(VERBOSE),)
  $(info INFO: ALL_SUBMAKE_STD_DATABYDIR_MULTIPLE: '$(ALL_SUBMAKE_STD_DATABYDIR_MULTIPLE)')
 endif

 # call the sub-make for each directory to process each "phony target", passing
 # the directory-specific make options that we calculated above.
 #- prev: v1: $(foreach t_targetphonyname,$(TARGETS_PHONY_RECURSIVE),$(strip \
 #- prev: v1:  )$(foreach t_dir,$(ALL_SUBMAKE_STD_DIRS),$(strip \
 #- prev: v1:   )$(eval $(strip \
 #- prev: v1:     $(t_targetphonyname) ::$(NEWLINE)$(strip \
 #- prev: v1:      )$(TAB)@$$(MAKE) $(strip \
 #- prev: v1:       )-C $(t_dir) $(strip \
 #- prev: v1:       )$(call FUNC_GET_SUBMAKE_STD_MAKEOPTS_FORDIR,$(t_dir)) $(strip \
 #- prev: v1:       )$$@ $(strip \
 #- prev: v1:       )$(NEWLINE)$(strip \
 #- prev: v1:    ))$(strip \
 #- prev: v1:   ))$(strip \
 #- prev: v1:  ))$(strip \
 #- prev: v1: ))
 #+ prev: v2:  (
 #+ prev: v2:     )-C $(t_dir) $(strip \
 #+ prev: v2:  )
 $(foreach t_targetphonyname,$(TARGETS_PHONY_RECURSIVE),$(strip \
  )$(foreach t_dir,$(ALL_SUBMAKE_STD_DIRS),$(strip \
   )$(eval $(strip \
    )$(t_targetphonyname) ::$(NEWLINE)$(strip \
     )$(TAB)$(CMD_SUBMAKE_START)$$(MAKE) $(strip \
      )$(call FUNC_GET_SUBMAKE_STD_MAKEOPTS_FORDIR,$(t_dir)) $(strip \
      )$$@ $(strip \
      )$(NEWLINE)$(strip \
   ))$(strip \
  ))$(strip \
 ))

 # create one rule for each of the files originally in
 # '$($(t_vn_pref)SUBMAKE_STD)' (now field values in
 # '$(ALL_SUBMAKE_STD_DATABYDIR_MULTIPLE)').
 #- prev: v1: $(foreach t_entry,$(strip \
 #- prev: v1:   )$(ALL_SUBMAKE_STD_DATABYDIR_MULTIPLE)$(strip \
 #- prev: v1:  ),$(strip \
 #- prev: v1:  )$(foreach t_targetphonyname,$(strip \
 #- prev: v1:    $(call FUNC_GET_SUBMAKE_STD_VALUEFROMENTRYFIELDNUM,$(strip \
 #- prev: v1:     )$(t_entry),$(strip \
 #- prev: v1:     )$(ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_PHONYTARGETNAME)$(strip \
 #- prev: v1:    ))$(strip \
 #- prev: v1:   )),$(strip \
 #- prev: v1:   )$(eval $(strip \
 #- prev: v1:     $(call FUNC_GET_SUBMAKE_STD_VALUEFROMENTRYFIELDNUM,$(strip \
 #- prev: v1:      )$(t_entry),$(strip \
 #- prev: v1:      )$(ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_OURTARGETFILE)$(strip \
 #- prev: v1:     ))$(strip \
 #- prev: v1:      ) :$(NEWLINE)$(strip \
 #- prev: v1:      )$(TAB)@$$(MAKE) $(strip \
 #- prev: v1:       )-C $(dir $$@) $(strip \
 #- prev: v1:       )$(call FUNC_GET_SUBMAKE_STD_MAKEOPTS_FORDIR,$(strip \
 #- prev: v1:         $(call FUNC_GET_SUBMAKE_STD_VALUEFROMENTRYFIELDNUM,$(strip \
 #- prev: v1:          )$(t_entry),$(strip \
 #- prev: v1:          )$(ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_DIR)$(strip \
 #- prev: v1:         ))$(strip \
 #- prev: v1:        ))$(strip \
 #- prev: v1:       )) $(strip \
 #- prev: v1:       )$(t_targetphonyname) $(strip \
 #- prev: v1:       )$(NEWLINE)$(strip \
 #- prev: v1:    ))$(strip \
 #- prev: v1:   ))$(strip \
 #- prev: v1:  ))$(strip \
 #- prev: v1: ))
 #+ prev: v2:  (
 #+ prev: v2:     )-C $(dir $$@) $(strip \
 #+ prev: v2:  )
 #? # NOTE: each target will depend on the same makefiles our local targets depend
 #? # on.  This is to work around the issue where a change in one of the makefiles
 #? # known to this makefile does not trigger a "remake" on the sub-make.
 #? # Alternatives such as having the sub-make target being a ".PHONY" target have
 #? # been discarded (not noticing when the file is absent, for example, is one of
 #? # the potential issues, along with "making too much" and thus always
 #? # re-generating the local target file(s).
 #+?  # ... : (
 #+?  )$(SRC_FILES_COMMON) $(strip \
 #+?   )
 $(foreach t_entry,$(strip \
   )$(ALL_SUBMAKE_STD_DATABYDIR_MULTIPLE)$(strip \
  ),$(strip \
  )$(foreach t_targetphonyname,$(strip \
    $(call FUNC_GET_SUBMAKE_STD_VALUEFROMENTRYFIELDNUM,$(strip \
     )$(t_entry),$(strip \
     )$(ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_PHONYTARGETNAME)$(strip \
    ))$(strip \
   )),$(strip \
   )$(eval $(strip \
    )$(call FUNC_GET_SUBMAKE_STD_VALUEFROMENTRYFIELDNUM,$(strip \
     )$(t_entry),$(strip \
     )$(ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_OURTARGETFILE)$(strip \
    ))$(strip \
     ) : $(strip \
      )$(ALL_SUBMAKE_STD_EXTRADEPS) $(strip \
      )$(NEWLINE)$(strip \
     )$(TAB)$(CMD_SUBMAKE_START)$$(MAKE) $(strip \
      )$(call FUNC_GET_SUBMAKE_STD_MAKEOPTS_FORDIR,$(strip \
        $(call FUNC_GET_SUBMAKE_STD_VALUEFROMENTRYFIELDNUM,$(strip \
         )$(t_entry),$(strip \
         )$(ALL_SUBMAKE_STD_DATABYDIR_FIELDNUM_DIR)$(strip \
        ))$(strip \
       ))$(strip \
      )) $(strip \
      )$(t_targetphonyname) $(strip \
      )$(NEWLINE)$(strip \
   ))$(strip \
  ))$(strip \
 ))

endif

# args:
# $(1): pre-command flags
# $(2): command
# $(3): post-command flags_args
define FUNC_SUDO_CMD
$(if $(filter 1 Y y YES yes TRUE true t,$(SUDO_NEEDED)),$(strip \
 )$(CMD_SUDO) $(strip \
  )$(strip $(1)) $(strip \
  )$(strip $(SUDO_OPTS_PRECMD)) $(strip \
  )$(strip $(2)) $(strip \
  )$(strip $(SUDO_OPTS_POSTCMD)) $(strip \
  )$(strip $(3)) $(strip \
  )$(strip \
 ),$(strip 
 )$(strip \
  )$(strip $(2)) $(strip \
  )$(strip $(3)) $(strip \
 )$(strip \
))
endef

# prev: v1: # args:
# prev: v1: # $(1): pre-command flags
# prev: v1: # $(2): command
# prev: v1: # $(3): post-command flags_args
# prev: v1: define FUNC_DOCKERCOMPOSE_CMD
# prev: v1: $(CMD_DOCKER_COMPOSE) $(strip \
# prev: v1:  )$(strip $(1)) $(strip \
# prev: v1:  )$(strip $(DOCKER_COMPOSE_OPTS_PRECMD)) $(strip \
# prev: v1:  )$(strip $(2)) $(strip \
# prev: v1:  )$(strip $(DOCKER_COMPOSE_OPTS_POSTCMD)) $(strip \
# prev: v1:  )$(strip $(3)) $(strip \
# prev: v1:  )
# prev: v1: endef

# args:
# $(1): pre-command flags
# $(2): command
# $(3): post-command flags_args
define FUNC_DOCKERCOMPOSE_CMD
$(call FUNC_SUDO_CMD,$(strip \
 )-H $(strip \
 )$(strip $(DOCKER_COMPOSE_SUDO_OPTS_PRECMD))$(strip \
 ),$(strip \
 )$(CMD_DOCKER_COMPOSE) $(strip \
  )$(strip $(1)) $(strip \
  )$(strip $(DOCKER_COMPOSE_OPTS_PRECMD)) $(strip \
  )$(strip $(2)) $(strip \
  )$(strip $(DOCKER_COMPOSE_OPTS_POSTCMD)) $(strip \
  )$(strip $(3)) $(strip \
  )$(strip \
 )$(strip \
))
endef

compose-up : files
	$(call FUNC_DOCKERCOMPOSE_CMD,up -d)

compose-stop : files
	$(call FUNC_DOCKERCOMPOSE_CMD,stop)

compose-rm : compose-stop files
	$(call FUNC_DOCKERCOMPOSE_CMD,rm -v)

.PHONY: $(TARGETS_PHONY)

# vi: set ts=3 sw=3 autoindent noexpandtab:
