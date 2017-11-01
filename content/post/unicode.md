---
featured_image: "/img/mackellar-american-printer-1882-pivotal-caster-1200rgb-2048x.jpg"
title: "UTF-8 all the way through"
description: "PostgreSQL speaks Unicode real well."
---

> PostgreSQL implements UTF-8 as a server encoding and as a client encoding,
> so that you can use unicode all the way through.

The default encoding and collation for a PostgreSQL database server are
setup at
[initdb](https://www.postgresql.org/docs/current/static/app-initdb.html)
time. This is usually done by your packaging system at installation.

# Database Encoding

It's important to set your environement correctly before installing
PostgreSQL, in order to avoid surprises. The following example sets the
system to use `en_US.UTF-8` before installing PostgreSQL on a docker
container using debian:

~~~
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get install -y locales
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen en_US.UTF-8 && \
    dpkg-reconfigure locales && \
    /usr/sbin/update-locale LANG=en_US.UTF-8
ENV LC_ALL en_US.UTF-8
~~~

To check your current installation settings, you can have a look at the
`template0` and `template1` databases settings:

~~~
\l template[01]
~~~

We can see that our local PostgreSQL *cluster* is using the intended server
side settings for unicode:

~~~
                                  List of databases
   Name    │  Owner   │ Encoding │   Collate   │    Ctype    │   Access privileges   
═══════════╪══════════╪══════════╪═════════════╪═════════════╪═══════════════════════
 template0 │ postgres │ UTF8     │ en_US.UTF-8 │ en_US.UTF-8 │ =c/postgres          ↵
           │          │          │             │             │ postgres=CTc/postgres
 template1 │ postgres │ UTF8     │ en_US.UTF-8 │ en_US.UTF-8 │ =c/postgres          ↵
           │          │          │             │             │ postgres=CTc/postgres
(2 rows)
~~~

When that's not the case, you need to destroy your setup and start from
scratch again. How exactly to do that depend on your distribution. Or maybe
on your Cloud provider.

# Server Encoding and Client Encoding

You can also have an *UTF8* encoded database and use a legacy application
(or programming language) that doesn't know how to handle Unicode properly.
In that case, you can ask PostgreSQL to convert all and any data on the fly
between the server-side encoding and your favorite client-side encoding,
thanks to the *client_encoding* setting.

~~~ sql
show client_encoding;
~~~

Here again, we use UTF8 client side, which allows handling French
accentuated characters we saw previously.

~~~ psql
 client_encoding 
═════════════════
 UTF8
(1 row)
~~~
Be aware that not all combinations of *server encoding* and *client
encoding* make sense. While it is possible for PostgreSQL to communicate
with your application using the *latin1* encoding on the client side if the
server side dataset includes texts in incompatible encodings, PostgreSQL
will issue an error. Such texts might be written using non-Latin scripts
such as Cyrillic, Chinese, Japanese, Arabic or other languages.

From the Emacs facility `M-x view-hello-file`, here's a table with spelling
of hello in plenty of different languages and scripts, all encoded in
*UTF8*:

~~~ psql
          language          │            hello        
════════════════════════════╪═════════════════════════════
 Czech (čeština)            │ Dobrý den
 Danish (dansk)             │ Hej / Goddag / Halløj
 Dutch (Nederlands)         │ Hallo / Dag
 English /ˈɪŋɡlɪʃ/          │ Hello
 Esperanto                  │ Saluton (Eĥoŝanĝo ĉiuĵaŭde)
 Estonian (eesti keel)      │ Tere päevast / Tere õhtust
 Finnish (suomi)            │ Hei / Hyvää päivää
 French (français)          │ Bonjour / Salut
 Georgian (ქართველი)        │ გამარჯობა
 German (Deutsch)           │ Guten Tag / Grüß Gott
 Greek (ελληνικά)           │ Γειά σας
 Greek, ancient (ἑλληνική)  │ Οὖλέ τε καὶ μέγα χαῖρε
 Hungarian (magyar)         │ Szép jó napot!
 Italian (italiano)         │ Ciao / Buon giorno
 Maltese (il-Malti)         │ Bonġu / Saħħa
 Mathematics                │ ∀ p ∈ world • hello p  □
 Mongolian (монгол хэл)     │ Сайн байна уу?
 Norwegian (norsk)          │ Hei / God dag
 Polish  (język polski)     │ Dzień dobry! / Cześć!
 Russian (русский)          │ Здра́вствуйте!
 Slovak (slovenčina)        │ Dobrý deň
 Slovenian (slovenščina)    │ Pozdravljeni!
 Spanish (español)          │ ¡Hola!
 Swedish (svenska)          │ Hej / Goddag / Hallå
 Turkish (Türkçe)           │ Merhaba
 Ukrainian (українська)     │ Вітаю
 Vietnamese (tiếng Việt)    │ Chào bạn
 Japanese (日本語)          │ こんにちは / ｺﾝﾆﾁﾊ
 Chinese (中文,普通话,汉语) │ 你好
 Cantonese (粵語,廣東話)    │ 早晨, 你好
~~~

Now, of course, I can't have that data sent to me in *latin1*:

~~~ psql
yesql# set client_encoding to latin1;
SET
yesql# select * from hello where language ~ 'Georgian';
ERROR:  character with byte sequence 0xe1 0x83 0xa5 in encoding "UTF8" has no equivalent in encoding "LATIN1"
yesql# reset client_encoding ;
RESET
~~~

So if it's possible for you, use *UTF-8* encoding and you'll have a much
simpler life. It must be noted that Unicode encoding makes comparing and
sorting text a rather costly operation. That said being fast and wrong is
not an option, so we are going to still use unicode text!
