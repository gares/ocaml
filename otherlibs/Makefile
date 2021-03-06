#**************************************************************************
#*                                                                        *
#*                                 OCaml                                  *
#*                                                                        *
#*            Xavier Leroy, projet Cristal, INRIA Rocquencourt            *
#*                                                                        *
#*   Copyright 1999 Institut National de Recherche en Informatique et     *
#*     en Automatique.                                                    *
#*                                                                        *
#*   All rights reserved.  This file is distributed under the terms of    *
#*   the GNU Lesser General Public License version 2.1, with the          *
#*   special exception on linking described in the file LICENSE.          *
#*                                                                        *
#**************************************************************************

# Common Makefile for otherlibs

ROOTDIR=../..
include $(ROOTDIR)/config/Makefile
include $(ROOTDIR)/Makefile.common

CAMLRUN ?= $(ROOTDIR)/boot/ocamlrun

ifeq "$(wildcard $(ROOTDIR)/flexdll/Makefile)" ""
export OCAML_FLEXLINK:=
else
export OCAML_FLEXLINK:=$(ROOTDIR)/boot/ocamlrun $(ROOTDIR)/flexdll/flexlink.exe
endif

CAMLC=$(CAMLRUN) $(ROOTDIR)/ocamlc -nostdlib -I $(ROOTDIR)/stdlib
CAMLOPT=$(CAMLRUN) $(ROOTDIR)/ocamlopt -nostdlib \
        -I $(ROOTDIR)/stdlib
OC_CFLAGS += $(SHAREDLIB_CFLAGS) $(EXTRACFLAGS)
OC_CPPFLAGS += -I$(ROOTDIR)/runtime

# Compilation options
COMPFLAGS=-absname -w +a-4-9-41-42-44-45-48 -warn-error A -bin-annot -g \
          -safe-string -strict-sequence -strict-formats $(EXTRACAMLFLAGS)
ifeq "$(FLAMBDA)" "true"
OPTCOMPFLAGS=-O3
else
OPTCOMPFLAGS=
endif
MKLIB=$(CAMLRUN) $(ROOTDIR)/tools/ocamlmklib

# Variables to be defined by individual libraries:
#LIBNAME=
#CLIBNAME=
#CMIFILES=
#CAMLOBJS=
#COBJS=
#EXTRACFLAGS=
#EXTRACAMLFLAGS=
#LINKOPTS=
#LDOPTS=
#HEADERS=

CMIFILES ?= $(CAMLOBJS:.cmo=.cmi)
CAMLOBJS_NAT ?= $(CAMLOBJS:.cmo=.cmx)
CLIBNAME ?= $(LIBNAME)

.PHONY: all allopt opt.opt # allopt and opt.opt are synonyms
all: lib$(CLIBNAME).$(A) $(LIBNAME).cma $(CMIFILES)

allopt: lib$(CLIBNAME).$(A) $(LIBNAME).cmxa $(LIBNAME).$(CMXS) $(CMIFILES)
opt.opt: allopt

$(LIBNAME).cma: $(CAMLOBJS)
	$(MKLIB) -o $(LIBNAME) -oc $(CLIBNAME) -ocamlc '$(CAMLC)' -linkall \
	         $(CAMLOBJS) $(LINKOPTS)

$(LIBNAME).cmxa: $(CAMLOBJS_NAT)
	$(MKLIB) -o $(LIBNAME) -oc $(CLIBNAME) -ocamlopt '$(CAMLOPT)' -linkall \
	         $(CAMLOBJS_NAT) $(LINKOPTS)

$(LIBNAME).cmxs: $(LIBNAME).cmxa lib$(CLIBNAME).$(A)
	$(CAMLOPT) -shared -o $(LIBNAME).cmxs -I . $(LIBNAME).cmxa

lib$(CLIBNAME).$(A): $(COBJS)
	$(MKLIB) -oc $(CLIBNAME) $(COBJS) $(LDOPTS)

install::
	if test -f dll$(CLIBNAME)$(EXT_DLL); then \
	  $(INSTALL_PROG) \
	    dll$(CLIBNAME)$(EXT_DLL) \
	    "$(INSTALL_STUBLIBDIR)/"; \
	fi
	$(INSTALL_DATA) lib$(CLIBNAME).$(A) "$(INSTALL_LIBDIR)/"
	cd "$(INSTALL_LIBDIR)"; $(RANLIB) lib$(CLIBNAME).$(A)

	$(INSTALL_DATA) \
	  $(LIBNAME).cma $(CMIFILES) \
	  "$(INSTALL_LIBDIR)/"
ifeq "$(INSTALL_SOURCE_ARTIFACTS)" "true"
	$(INSTALL_DATA) \
	  $(CMIFILES:.cmi=.mli) \
          $(CMIFILES:.cmi=.cmti) \
	  "$(INSTALL_LIBDIR)/"
endif
	if test -n "$(HEADERS)"; then \
	  $(INSTALL_DATA) $(HEADERS) "$(INSTALL_LIBDIR)/caml/"; \
	fi

installopt:
	$(INSTALL_DATA) \
	   $(CAMLOBJS_NAT) $(LIBNAME).cmxa $(LIBNAME).$(A) \
	   "$(INSTALL_LIBDIR)/"
	cd "$(INSTALL_LIBDIR)"; $(RANLIB) $(LIBNAME).a
	if test -f $(LIBNAME).cmxs; then \
	  $(INSTALL_PROG) $(LIBNAME).cmxs "$(INSTALL_LIBDIR)/"; \
	fi

partialclean:
	rm -f *.cm*

clean:: partialclean
	rm -f *.dll *.so *.a *.lib *.o *.obj

.SUFFIXES: .ml .mli .cmi .cmo .cmx .$(O)

.mli.cmi:
	$(CAMLC) -c $(COMPFLAGS) $<

.ml.cmo:
	$(CAMLC) -c $(COMPFLAGS) $<

.ml.cmx:
	$(CAMLOPT) -c $(COMPFLAGS) $(OPTCOMPFLAGS) $<

.c.$(O):
	$(CC) -c $(OC_CFLAGS) $(OC_CPPFLAGS) $(OUTPUTOBJ)$@ $<
