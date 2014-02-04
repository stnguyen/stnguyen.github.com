---
layout: post
title: "Install Moses on Ubuntu 10.10"
date: 2012-01-19 11:24:00
comments: true
categories: natural-language-processing
tags: nlp smt moses ubuntu install
---

Several months ago I had a chance to work on a Machine-translation project in which we use [Moses](http://www.statmt.org/moses/) for Statistical machine translation (SMT).

I had a hard time try to figure out how to make Moses run properly on my system (Ubuntu 10.10), since I was new to Linux OS at that time. This is a note I wrote for myself of how I did successfully install Moses on my machine.

Hope it will help someone ;)

Prepare folders
---

I'll install moses into my home directory

    $ mkdir ~/moses
    $ mkdir ~/moses/downloads
    $ mkdir ~/moses/bin

Downloading
---

Download giza, irstlm, moses and additional scripts into downloads directory:

    $ wget http://giza-pp.googlecode.com/files/giza-pp-v1.0.3.tar.gz
    $ svn co https://irstlm.svn.sourceforge.net/svnroot/irstlm irstlm_svn
    $ svn co https://mosesdecoder.svn.sourceforge.net/svnroot/mosesdecoder/trunk moses_svn
    $ wget http://homepages.inf.ed.ac.uk/jschroe1/how-to/scripts.tgz

Installing
---

### giza++ and mkcls

    $ tar zxf giza-pp-v1.0.3.tar.gz
    $ cd giza-pp
    $ make

Giza don't have `make install`, so let's copy compiled files manually

    $ cp GIZA++-v2/GIZA++ ~/moses/bin/
    $ cp mkcls-v2/mkcls ~/moses/bin/
    $ cp GIZA++-v2/snt2cooc.out ~/moses/bin/
    $ cp GIZA++-v2/snt2plain.out ~/moses/bin/

### irstlm

    $ cd ../irstlm_svn
    $ ./check_version.sh
    $ ./regenerate-makefiles.sh
    $ ./configure --prefix=/home/[your_user_name]/moses/bin/irstlm
    $ make
    $ make install

### moses

    $ cd ../moses_svn
    $ ./regenerate-makefiles.sh

If you encounter some error like this:

    Calling /usr/bin/aclocal...
    Calling /usr/bin/autoconf...
    Calling /usr/bin/automake...
    OnDiskPt/src/Makefile.am:1: library used but `RANLIB' is undefined
    OnDiskPt/src/Makefile.am:1:   The usual way to define `RANLIB' is to add `AC_PROG_RANLIB'
    OnDiskPt/src/Makefile.am:1:   to `configure.in' and run `autoconf' again.
    kenlm/Makefile.am:1: Libtool library used but `LIBTOOL' is undefined
    kenlm/Makefile.am:1:   The usual way to define `LIBTOOL' is to add `AC_PROG_LIBTOOL'
    kenlm/Makefile.am:1:   to `configure.in' and run `aclocal' and `autoconf' again.
    kenlm/Makefile.am:1:   If `AC_PROG_LIBTOOL' is in `configure.in', make sure
    kenlm/Makefile.am:1:   its definition is in aclocal's search path.
    moses-chart/src/Makefile.am:1: library used but `RANLIB' is undefined
    moses-chart/src/Makefile.am:1:   The usual way to define `RANLIB' is to add `AC_PROG_RANLIB'
    moses-chart/src/Makefile.am:1:   to `configure.in' and run `autoconf' again.
    moses/src/Makefile.am:1: Libtool library used but `LIBTOOL' is undefined
    moses/src/Makefile.am:1:   The usual way to define `LIBTOOL' is to add `AC_PROG_LIBTOOL'
    moses/src/Makefile.am:1:   to `configure.in' and run `aclocal' and `autoconf' again.
    moses/src/Makefile.am:1:   If `AC_PROG_LIBTOOL' is in `configure.in', make sure
    moses/src/Makefile.am:1:   its definition is in aclocal's search path.
    automake failed

Try to install libtool: `sudo apt-getinstall libtool` then re-run regenerate-makefiles.sh

    $ ./configure --with-irstlm=/home/[your_user_name]/moses/bin/irstlm
    $ make

### Mosese support scripts

    $ cd scripts

Edit Makefile to supply new values for TARGETDIR and BINDIR

    TARGETDIR=/home/stefan/moses/bin/moses_scripts
    BINDIR=/home/stefan/moses/bin

Now run:

    $ make release

And you're done.

### More resources

- To learn how to train models, have a look at [http://www.statmt.org/moses/?n=FactoredTraining.HomePage](http://www.statmt.org/moses/?n=FactoredTraining.HomePage)
- To try out Moses with a sample trained model, follow [http://www.statmt.org/moses/?n=Moses.Tutorial](http://www.statmt.org/moses/?n=Moses.Tutorial)
- There’re also a baseline guiding you from preparing data, training models to tuning, testing and evaluating models: [http://www.statmt.org/wmt11/baseline.html](http://www.statmt.org/wmt11/baseline.html)
