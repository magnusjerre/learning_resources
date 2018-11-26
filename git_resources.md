# Git tips and tricks!

## .gitattributes
Brukes for å håndtere crlf og eol per repository, dette burde brukes fremfor å være avhenig av at alle som er en del av prosjektet skal sette riktige globale verdier. Den siste linjen som pattern matches er den gjeldende.

```
*.filetype binary	//Betyr at filen skal behandles som binær og vil derfor ikke være diff-able
```
Ved å sette spesfikke filer/filtyper til å være binære, vil de ikke git vise diff for disse filene, hverken med **git diff** eller **git show**. Git vil heller ikke prøve seg på crlf fiks.

### text
Dette attributet enabler og kontrollerer end-of-line normalisering. Ved normalisering vil filene gjøres til LF når de sendes til repositoriet. For å endre hvordan det er i "working directory" bruk **eol**.
```
*.filtetype text	//Markerer filen som en tekstfil og enabler end-of-line normalisering
*.filteype -text	//Ikke bruk end-of-line normalisering
*.filtype text=auto	//La git finne ut om end-of-line normalisering skal brukes. Fint å ha som default på første linje i gitattributes filen.
```
### eol
Dette attributtet setter en spesifikk line-ending som skal brukes i "working directory".
```
*.filtype eol=crlf	//Normaliser line endings ved checkin og konverter til crlf ved checkout (bra for windows)
*.filtype eol=lf	//Normaliser ved checkin, men IKKE konverter til crlf ved checkout (bra for unix)
```

# Git workshop
- list alle config-variabler: `git config -l`
- fetch alle brancher: `git fetch --all`
- rebase pull: `git pull -r`, kan eventuelt gjøre `git pull -r=interactive`
- slette branch remote vha ":": `git push origin :branch_navn`
