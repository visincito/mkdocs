---
tags: 
  - SIP
  - VoIP
---
# Google TTS script for Asterisk

This script makes use of Google's translate text to speech service
in order to render text to speech and play it back to the user.
It supports a variety of different languages.

This TTS service is 'unofficial' and not supported by Google,
it can be terminated at any point with no warning.
People looking for TTS solutions to base their projects/products on
should look for alternative, officially supported  services.

## Requirements

- Perl --- (**The Perl Programming Language**)
- perl-libwww  --- (**The World-Wide Web library for Perl**)
- perl-LWP-Protocol-https ---  (**For HTTPS support**)
- sox ---  (**Sound eXchange, sound processing program**)
- mpg123 ---  (**MPEG Audio Player and decoder**)
- Internet access in order to contact google and get the voice data.

## Installation

[Repository](https://github.com/visincito/asterisk-googletts)

To install copy googletts.agi to your agi-bin directory.
Usually this is /var/lib/asterisk/agi-bin/
To make sure check your /etc/asterisk/asterisk.conf file

### Usage

``` title="Syntaxys In Dialplan"
agi(googletts.agi,"text",[language],[intkey],[speed]) 
```

This will invoke the Google TTS engine, render the text string to speech and play it back to the user.
If 'intkey' is set the script will wait for user input. Any given interrupt keys will
cause the playback to immediately terminate and the dialplan to proceed to the
matching extension (this is mainly for use in IVR, see README for examples).
If 'speed' is set the speech rate is altered by that factor (defaults to 1.2).

The script contacts Google's TTS service in order to get the voice data
which then stores in a local cache (by default /tmp/) for future use.
Parameters like default language, enabling or disabling caching and cache dir
can be set up by editing the script.

## Examples

``` javascript  title="sample dialplan code for your extensions.conf"

;GoogleTTS Demo
;PLayback messages to user

exten => 1234,1,Answer()
    ;;Play mesage in English:
exten => 1234,n,agi(googletts.agi,"This is a simple google text to speech test in english.",en)
    ;;Play message in Spanish
exten => 1234,n,agi(googletts.agi,"Esta es una simple prueba en español.",es)
    ;;Play message in Greek
exten => 1234,n,agi(googletts.agi,"Αυτό είναι ένα απλό τέστ στα ελληνικά.",el)

;A simple dynamic IVR using GoogleTTS"
[my_ivr]

exten => s,1,Answer()
exten => s,n,Set(TIMEOUT(digit)=5)
exten => s,n,agi(googletts.agi,"Welcome to my small interactive voice response menu.",en)
    ;;Wait for digit:
exten => s,n(start),agi(googletts.agi,"Please dial a digit.",en,any)
exten => s,n,WaitExten()

    ;;PLayback the name of the digit and wait for another one:
exten => _X,1,agi(googletts.agi,"You just pressed ${EXTEN}. Try another one please.",en,any)
exten => _X,n,WaitExten()

exten => i,1,agi(googletts.agi,"Invalid extension.",en)
exten => i,n,goto(s,start)

exten => t,1,agi(googletts.agi,"Request timed out.",en)
exten => t,n,goto(s,start)

exten => h,1,Hangup()
```

### Supported Languages

??? note "Languages Code List"
    |**Language**|**Code**  |
    |:---------|:-----------|
    |Afrikaans:          |af|
    |Albanian:           |sq|
    |Amharic:            |am|
    |Arabic:             |ar|
    |Armenian:           |hy|
    |Azerbaijani:        |az|
    |Basque:             |eu|
    |Belarusian:         |be|
    |Bengali:            |bn|
    |Bihari:             |bh|
    |Bosnian:            |bs|
    |Breton:             |br|
    |Bulgarian:          |bg|
    |Cambodian:          |km|
    |Catalan:            |ca|
    |Chinese Simplified: |zh-CN|
    |Chinese Traditional:|zh-TW|
    |Corsican:           |co|
    |Croatian:           |hr|
    |Czech:              |cs|
    |Danish:             |da|
    |Dutch:              |nl|
    |English:            |en|
    |Esperanto:          |eo|
    |Estonian:           |et|
    |Faroese:            |fo|
    |Filipino:           |tl|
    |Finnish:            |fi|
    |French:             |fr|
    |Frisian:            |fy|
    |Galician:           |gl|
    |Georgian:           |ka|
    |German:             |de|
    |Greek:              |el|
    |Guarani:            |gn|
    |Gujarati:           |gu|
    |Hausa:              |ha|
    |Hebrew:             |iw|
    |Hindi:              |hi|
    |Hungarian:          |hu|
    |Icelandic:          |is|
    |Indonesian:         |id|
    |Interlingua:        |ia|
    |Irish:              |ga|
    |Italian:            |it|
    |Japanese:           |ja|
    |Javanese:           |jw|
    |Kannada:            |kn|
    |Kazakh:             |kk|
    |Kinyarwanda:        |rw|
    |Kirundi:            |rn|
    |Korean:             |ko|
    |Kurdish:            |ku|
    |Kyrgyz:             |ky|
    |Laothian:           |lo|
    |Latin:              |la|
    |Latvian:            |lv|
    |Lingala:            |ln|
    |Lithuanian:         |lt|
    |Macedonian:         |mk|
    |Malagasy:           |mg|
    |Malay:              |ms|
    |Malayalam:          |ml|
    |Maltese:            |mt|
    |Maori:              |mi|
    |Marathi:            |mr|
    |Moldavian:          |mo|
    |Mongolian:          |mn|
    |Montenegrin:        |sr-ME|
    |Nepali:             |ne|
    |Norwegian:          |no|
    |Norwegian Nynorsk:  |nn|
    |Occitan:            |oc|
    |Oriya:              |or|
    |Oromo:              |om|
    |Pashto:             |ps|
    |Persian:            |fa|
    |Polish:             |pl|
    |Portuguese:         |pt|
    |Portuguese Brazil:  |pt-BR|
    |Portuguese Portugal:|pt-PT|
    |Punjabi:            |pa|
    |Quechua:            |qu|
    |Romanian:           |ro|
    |Romansh:            |rm|
    |Russian:            |ru|
    |Scots Gaelic:       |gd|
    |Serbian:            |sr|
    |Serbo-Croatian:     |sh|
    |Sesotho:            |st|
    |Shona:              |sn|
    |Sindhi:             |sd|
    |Sinhalese:          |si|
    |Slovak:             |sk|
    |Slovenian:          |sl|
    |Somali:             |so|
    |Spanish:            |es|
    |Sundanese:          |su|
    |Swahili:            |sw|
    |Swedish:            |sv|
    |Tajik:              |tg|
    |Tamil:              |ta|
    |Tatar:              |tt|
    |Telugu:             |te|
    |Thai:               |th|
    |Tigrinya:           |ti|
    |Tonga:              |to|
    |Turkish:            |tr|
    |Turkmen:            |tk|
    |Twi:                |tw|
    |Uighur:             |ug|
    |Ukrainian:          |uk|
    |Urdu:               |ur|
    |Uzbek:              |uz|
    |Vietnamese:         |vi|
    |Welsh:              |cy|
    |Xhosa:              |xh|
    |Yiddish:            |yi|
    |Yoruba:             |yo|
    |Zulu:               |zu|

### Security Considerations

This script contacts Google servers in order to get voice data.
The script uses TLS to encrypt all the traffic between your pbx and these servers
so no 3rd party can eavesdrop your communication, but your data will be available
to Google under a not yet defined policy.

## Tiny version

The '-tiny' suffixed scripts use the HTTP::Tiny perl module instead of LWP::UserAgent.
This makes them a lot faster when run in small embedded systems or boards like
the Raspberry pi. They can be used as an in-place replacement of the normal versions
of the TTS scripts and expose the same interface/cli args. To use them just make sure
you have HTTP::Tiny installed.
In debian or ubuntu based distros: 'apt-get install libhttp-tiny-perl'
In distros that don't have it in their repos: 'cpan HTTP::Tiny'
It also requires mp3 format support in sox in order to avoid the use of mpg123.

## License

The GoogleTTS script for asterisk is distributed under the GNU General Public
License v2. See COPYING for details.

### Homepage

[http://zaf.github.com/asterisk-googletts/](http://zaf.github.com/asterisk-googletts/)

Fork repository:

[https://github.com/visincito/asterisk-googletts](https://github.com/visincito/asterisk-googletts)
