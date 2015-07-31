# R-fortunes

This contains fortunes from the following standard linux `fortunes` packages,
 converted into a format that the [R fortunes package](https://cran.r-project.org/web/packages/fortunes/index.html) understands.
Just about all the `fortunes-*` packages except for `fortunes-off`[ensive], and language-specific ones.
See [Fortunes List][] for a more detailed list.

## Instructions

(**Note - currently not working, something about embedded newlines and the way `read.fortunes()` reads stuff in. Currently I patch my .Rprofile as described in [R fortune monkey-patching][]**).

Just place the CSV files of the fortunes you want into `system.file('fortunes', package='fortunes')` (usually `/path/to/R/library/fortunes/fortunes`). Then the `fortunes` package will automagically read them

```r
library(fortunes)
fortune()
```

Currently there is not a way to show fortunes from some subset of the sources only - you'd have to write a wrapper e.g.

```r
# example: my.fortune(c('men-women.dat', 'fortunes-min'))
my.fortune <- function (sources) {
    # crude regex construction
    source.regexp <- sprintf('\\b(%s)\\b', gsub('.', '\\.', paste(sources, collapse='|'), fixed=T))
    fortune(fortunes.data=subset(fortunes:::fortunes.env$fortunes.data, grepl(source.regexp, source)))
}
```

The sources are listed as ('{filename} ({package name})', where '{filename}' and '{package name}' are the dat file and package name respectively (e.g. 'art.dat' and 'fortunes')

## Fortunes List

* `fortunes-bofh-excuses`: bofh-excuses.dat
* `fortunes-debian-hints`: debian-hints.dat
* `fortunes-mario` (**note** - need to adjust locale for some of these)
  * mario.anagramas.dat
  * mario.arteascii.dat
  * mario.computadores.dat
  * mario.gauchismos.dat
  * mario.geral.dat
  * mario.palindromos.dat
  * mario.piadas.dat
* `fortunes-min`:
  * fortunes.dat
  * literature.dat
  * riddles.dat
* `fortunes-ubuntu-server`: ubuntu-server-tips.dat
* `fortunes`:
  * art.dat
  * ascii-art.dat
  * computers.dat
  * cookie.dat
  * debian.dat
  * definitions.dat
  * disclaimer.dat
  * drugs.dat
  * education.dat
  * ethnic.dat
  * food.dat
  * goedel.dat
  * humorists.dat
  * kids.dat
  * knghtbrd.dat
  * law.dat
  * linuxcookie.dat
  * linux.dat
  * love.dat
  * magic.dat
  * medicine.dat
  * men-women.dat
  * miscellaneous.dat
  * news.dat
  * paradoxum.dat
  * people.dat
  * perl.dat
  * pets.dat
  * platitudes.dat
  * politics.dat
  * science.dat
  * songs-poems.dat
  * sports.dat
  * startrek.dat
  * tao.dat
  * translate-me.dat
  * wisdom.dat
  * work.dat
  * zippy.dat

# Details

## R fortunes CSV

For those that are interested.

An R fortunes file must have CSV extension and be in the `system.file('fortunes', package='fortunes')` directory.
Despite being a C(omma)-separated values file, the separator is ';'.
The header is "quote;author;context;source;date".

The files are read in with this command in `read.fortunes()` (which we cannot adjust):

    read.table(file, header = TRUE, sep = ";", 
               quote = "\"", colClasses = "character"))

At the moment (31 Jul) I'm having trouble with multiline fortunes, even with quoting, so I'll tinker around and see.

## R fortune monkey-patching

For now, I monkey patch the fortunes DB. The fortunes database is stored in

    fortunes:::fortunes.env$fortunes.data

If you modify this it's used automagically.

Out of interest, the script I used to generate these CSVs/monkey patch R fortunes is here:

```r
patch.fortunes <- function (fortune.directory='/usr/share/games/fortunes', use.iconv=T) {
    if (!require(fortunes, quietly=T)) {
        return()
    }

    source.files = rbind(
     data.frame(source='fortunes-bofh-excuses',
                files='bofh-excuses.dat',
                stringsAsFactors=F),
     data.frame(source='fortunes-debian-hints',
                files='debian-hints.dat',
                stringsAsFactors=F),
     data.frame(source='fortunes-mario',
                files=c('mario.anagramas.dat',
                        'mario.arteascii.dat', 'mario.computadores.dat',
                        'mario.gauchismos.dat', 'mario.geral.dat', 'mario.palindromos.dat',
                        'mario.piadas.dat'),
                stringsAsFactors=F),
     data.frame(source='fortunes-min',
                files=c('fortunes.dat', 'literature.dat', 'riddles.dat'),
                stringsAsFactors=F),
     data.frame(source='fortunes-ubuntu-server', files='ubuntu-server-tips.dat', stringsAsFactors=F),
     data.frame(source='fortunes',
                files=c('art.dat', 'ascii-art.dat', 'computers.dat', 'cookie.dat',
                        'debian.dat', 'definitions.dat', 'disclaimer.dat', 'drugs.dat',
                        'education.dat', 'ethnic.dat', 'food.dat', 'goedel.dat',
                        'humorists.dat', 'kids.dat', 'knghtbrd.dat', 'law.dat',
                        'linuxcookie.dat', 'linux.dat', 'love.dat', 'magic.dat',
                        'medicine.dat', 'men-women.dat', 'miscellaneous.dat',
                        'news.dat', 'paradoxum.dat', 'people.dat', 'perl.dat',
                        'pets.dat', 'platitudes.dat', 'politics.dat', 'science.dat',
                        'songs-poems.dat', 'sports.dat', 'startrek.dat', 'tao.dat',
                        'translate-me.dat', 'wisdom.dat', 'work.dat', 'zippy.dat'),
                stringsAsFactors=F)
    )
    source.file.map <- source.files$source
    names(source.file.map) <- sub('\\.dat$', '', source.files$files)

    # ----- extract
    files = list.files(fortune.directory, recursive=F, pattern='^[^.]+$')

    # Fortunes separated by newline with '%'
    f <- do.call(rbind,
      c(lapply(files, function (f) {
           l <- readLines(file.path(fortune.directory, f), warn=F)
           seps <- unique(c(1, which(l == '%'), length(l)))
           l[seps] <- ''
           l <- unlist(tapply(l, cut(seq_along(l), seps), paste, collapse='\n', simplify=F), use.names=F) # exclude last el of '%'
           # bylines
           # \n\t\t-- Author
           # (star trek) \n {spaces} [person[, year]]
           # hrm be smarter and look @ last line?
           # blah, some go over multiple lines! join & split back up?
           #gsub('\n( +)\\[(?=.+\\]$|(?=[^\\]\n]+\n\\1[^ ].*
           if (use.iconv) l <- iconv(l, to='ASCII', sub='')
           l <- strsplit(l, '\n\t\t-- ' )
           l <- data.frame(quote=gsub('\n$|^\n', '', vapply(l, '[', i=1, 'template')),
                      author=sub('\n$', '', vapply(l, '[', i=2, 'template')),
                      context=NA,
                      source=sprintf('%s.dat (%s)', f, source.file.map[f]),
                      date=NA,
                      stringsAsFactors=F)
           # Note: author is **required**
           l$author[is.na(l$author)] <- sprintf('Unknown (%s)', file.path(fortune.directory, f))

           # write out the fortunes (note - embedded newlines problem, make sure
           # files can be read in by
           # `read.table(file, header=T, sep=';', quote='"', colClasses='character')`
           # ofile <- sprintf('~/r-fortunes/fortunes/%s.csv', f)
           # write.table(l, file=ofile, sep=';', row.names=F, quote=T)
           l
      })
    , make.row.names=F, stringsAsFactors=F))

    # try extract dates?...nah, can't BB

    # ------ patch
    # environments are by reference...
    x = fortunes:::fortunes.env
    if (is.null(x$fortunes.data))
        x$fortunes.data <- read.fortunes()

    if (!exists('.PATCHED_FORTUNES', envir=x, mode='logical') || !x$.PATCHED_FORTUNES) {
        x$.PATCHED_FORTUNES <- T
        x$fortunes.data <- rbind(x$fortunes.data, f)
    }
}
```

Then I put this into my `.Rprofile` to patch the fortunes (for this session) and give a startup fortune.

```r
if (require(fortunes, quietly=T)) {
    patch.fortunes()
    fortune()
}
```

There is a small but noticeable delay on startup, while the fortunes files are processed.
Ideally I work out how to generate fortune CSVs that can be placed in the `fortunes/` directory
 so that R doesn't have to re-read all the files each time.

# Other fun

I recommend the [cowsayr package](https://github.com/mathematicalcoffee/cowsay) so that you can have a cow speak your fortune!
