# LSD Eksamens Rapport

## Udviklere
> - Kristjan Reinert Gásadal
> - Lasse René Hansen
> - Martin Lukas Hansen
> - Michael Boje Veilis

**Afleveret:** 21/12/2018

## Introduktion
Larges System Development kurset har haft fokus på hvordan man etablerer et udviklings- og produktionsmiljø med Continuous Integration og Continuous Delivery.<br />
Formålet var at implementere en kopi af Hackernews hjemmesiden: https://news.ycombinator.com/
 
Forløbet bestod af to perioder, hvor første periode var udvikling af vores projekt og anden periode bestod af at observere en anden gruppes projekt.<br />
Redskaberne brugt i udviklings perioden var meget selvvalgt og ud fra hvad vi syntes var nemmest, hvor i perioden vi skulle observere den anden gruppes projekt, blev vi introduceret til forskellige 3rd-party redskaber der skulle tages i brug for bedre observering og rapportering af problemer. Alt forklaret mere uddybende i denne rapport.

## 1. Krav, arkitektur, design og process
Vi kommer i dette afsnit ind på vores Krav, arkitektur af hele vores system, design af de forskellige komponenter og den process vi har brugt for at lave dette projekt.

### 1.1. System krav
Systemet havde følgende minimums krav til funktionalitet:

1. Display a set of stories on your system's front page.
2. Display a set of comments on stories.
3. Stories or comments are posted by users, which have to be registered to and logged into the system to be able to post.
4. Users login to the system via a separate page.
5. Users are identified by a user name and a password.
6. New users can register to the system, via a separate page.
7. The complete HTTP API, as defined below.

- Accept posted stories/comments/etc.
- Provide the id of the latest digested post.
- Provide status information.

### 1.2. Udviklings process

Der var krav om at udviklingen skulle ske via scrum.

``(U-01) As a User I want to be able to register so that I can post new discussions as well as comments``

Rollerne fra Scrum, som product owner, Scrum master, technical lead osv., lå forholdsvis løst, hvor ansvaret for de forskellige områder var fælles. Sprints bestod af én uges udviklingsforløb, efterfulgt af et retrospektiv samt et technical review med underviseren (Helge). Forløbet var delvist styret af det faktum, at der var løbende afleveringer, hvor der var en feature eller et dokument, som skulle være færdig inden for en vis tidsperiode.  

Til et scrum forløb hører der selvfølgelig en product- og sprint backlog. Vores backlog kan findes her og vores individuelle kanban board kan findes her. Vores backlog består af en opdeling af user stories i deres respektive sprint periode. Herunder ses den generelle struktur:

![Scrumboard](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/Scrumboard.png)

Det gav i øvrigt mening at udnytte en række af principperne fra XP - der fint komplimenterer Scrum. Dette inkluderer, men begrænser sig ikke til, Pair Programming til de mere udfordrende opgaver. Simpelt design var i fokus, da koden helst skal være overskuelig, selv hvis det set udefra. Testing er vigtigt mht. at garantere en vis kvalitet. Kontinuerlig integration(Travis CI) anvendte vi til at sikre os, at der ikke blev pushet en masse kode op, som skabte en masse uforventede fejl. Koden blev refaktoreret efter behov og vi blev i starten af forløbet enige om en fælles kodningsstandard.

Ud over projekt management anvendte vi også Github som versionsstyring, hvor vi netop kunne lave regelmæssige pushes og oprette branches efter behov mht. teknologiske spikes. Her udgav vi vores ugentlige releases, som led i forløbets afleveringer. Vores samlede projekt bestod af flere github repositories, til henholdsvis udvikling, eksperimentel kode, dokumentation, opsætning mm.

### 1.3. Software arkitektur
Softwarearkitekturen i vores projekt består overordnet af en spartansk frontend, udviklet med ren Javascript, HTML og CSS, en backend lavet med Java, JDBC og JavaX.WS.RS samt en Database lavet med MySQL.

Som Cloud-udbyder valgte vi at bruge DigitalOcean. Vi overvejede ikke at bruge en anden cloud service. Vi bestilte en droplet, som kørte hele projektet. Der blev også lavet en droplet til vores overvågningssystemer, Prometheus og Grafana, hvis funktion var at overvåge vores system for mulige fejl. Setuppet endte med at se sådan her ud:

![Arkitektur1](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/Arkitektur1.png)
![Arkitektur2](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/Arkitektur2.png)

Den ene droplet kørte vores Frontend, Backend og Database, hvor den anden droplet kørte vores Prometheus og Grafana server.

Her ses en overordnet arkitektur af systemet og håndtering af data:

![Arkitektur3](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/Arkitektur3.png)

Der er fire war filer til de forskellige funktionaliteter. Status svarer enten “Alive”, “Update”, “Down”. Frontend er HTML, CSS og JS. Base tager imod data og Latest giver data i form af JSON.

### 1.4. Software design
Da vi fik projektet diskuterede vi og undersøgte hvordan dette projekt kunne implementeres samt hvilke redskaber der skulle bruges. Så vidt vi vidste skulle man bruge java, men det var åbenbart ikke tilfældet. Vi blev tildelt et dokument om hvilke software kravspecifikationer der skulle laves på både i de non-funktionelle og funktionelle krav. Der blev også set på hvordan Hackernews hjemmesiden så ud og hvordan det officielle API så ud inden vi begyndte at designe vores projekt. Efter at have gennemskuet hvordan vi ville lave det ud fra den information vi nu kunne finde og blev tildelt, kunne vi så designe vores Frontend, Backend og Database.

#### Frontend
Vores frontend blev bygget med JS, HTML og CSS og benytter sig ikke af nogle frameworks og virker på en iPhone 4, som ikke kan ES6. Man kunne også have brugt NodeJS til at servicere frontend men så ville det kræve at tillade Cross-origin resource sharing da det ikke kan dele port med java serveren. Det ville også være ekstra ueffektivt at have to garbage collectede sprog kørende samtidigt på samme maskine. Herunder ses hvordan designet ser ud og hvordan vores frontend fungerer.

![Front1](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/Front1.png)
![Front2](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/Front2.png)

Ved at trykke på en post som en bruger har lavet, kan siden hoppe videre til den tråd som posten står i, og hoppe direkte hen til den post hvor end den der.

![Front3](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/Front3.png)

Her ses en tråd der er loadet. Det meste af tiden der går på at loade en side er konvertering fra Database → JSON og på client side JSON → HTML. Bid mærke i at HTML tags i posts escapes, så folk ikke kan lave alt for sjove ting på siden.

![Front4](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/Front4.png)

#### Backend
Vores backend blev lavet i Java via JavaX.WS.RS og kører på en WildFly server. Der var lidt spekulering i starten om vi ville prøve at lave vores webservice med Spring-boot frameworket, men besluttede ikke at gøre det alligevel da ingen af os havde erfaringer med det. Vi tænkte at performance af databaseforbindelserne var vigtigt og fandt noget der hedder HikariCP som er en connection pool. Ifølge deres egen hjemmeside er de den hurtigeste connection pool til Java. Den var nem at bruge og at sætte op. For at oprette en forbindelse skulle der kun laves noget konfiguration i forbindelsesklassen og indsættes et dependency i POM filen. Siden projektet også var et webprojekt blev der også lavet REST-API’er, da det bestod af flere komponenter, som backend og frontend, der skulle snakke sammen.

#### Database
Al data persisteres i en MySQL database. Databasestrukturen er lavet til at være så direkte kompatibel med Helges simulatordata som muligt. Tabellen ratings blev ikke aktuel.

![DB-billede1](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/DB-billede1.png)

![DB-billede2](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/DB-billede2.png)
![DB-billede3](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/DB-billede3.png)
![DB-billede4](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/DB-billede4.png)
![DB-billede5](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/DB-billede5.png)

Helges simulatordata er så upraktisk lavet at der ikke gemmes et trådid i hver post. Der står kun hvilken post en post er et svar til. Hvis man vil bruge den databasestruktur skal man lave rekursive databaseopslag for at læse en hel tråd. Vi troede at det skulle være realistisk så man havde en service man ville bruge. I sådan et tilfælde er det langsomt at lave rekursive databaseopslag, så vi tilføjede et ekstra trådid-felt som nedarves ved indsættelse af data. Ved indsættelse af en ny tråd er dette felt lig postid, og ellers nedarver man fra parent. Vores gruppe er en af de få grupper der har implementeret at man rent faktisk kan lave opslag på tråde. Desuden er vores frontend til at læse tråde langt mere genialt end det der er på HackerNews. På HackerNews vises kommentarer som træer. Det betyder at når der kommer nye posts er det utroligt besværligt at finde ud af hvilke posts der er nye, for de nye posts kan være hvor som helst på skærmen. I vores design er posts altid sorteret efter postid, og de vises ikke som træer. I stedet er der backlinks til det en post er et svar til.

**Insert-performance**
Vi brugte Helges student_tester.py til at benchmarke servicen. Benchmarket blev kørt på Lasses bærbar. Helge havde sat timeouten som default til 0.1 sek og påstod at tiden i timeout ikke ville påvirke hvor hurtigt sriptet kører, men det er usandt. Ved at køre scriptet med forskellige timeouts tog det hhv kortere eller længere tid at køre scriptet - selv om alle requests blev indsat. Det viste sig at være ret langsomt at indsætte posts en ad gangen. Det kunne tage op mod 0,4 sek at indsætte en enkel post. Det meste af tiden går dog bare på at oprette en forbindelse til MySQL og at skrive data ned på disken. En harddisk skal seeke et sted hen for så at kunne skrive sekventielt. Det er hurtigt at læse eller skrive sekventielt, men at seeke er meget langsomt. Vi prøvede en anden arkitektur med samme benchmark som viste sig at køre langt hurtigere. Posts bliver taget imod med det samme, og indsat i en linked list. En baggrundstråd tager hvad end der er i listen hvert sekund og indsætter det i databasen - flere elementer ad gangen. Som det fremgår af resultaterne nedenunder, så var det en markant forbedring. Helge viste på et tidspunkt en graf over hvor mange posts der ville komme, som havde spikes op til omkring 1000. Vi kunne ikke huske hvad tidsenheden skulle være, og frygtede at det ville være i sekunder. Forestil jer vores overraskelse da det viste sig at være i minuttet, ikke i sekundet, når vi havde lavet noget med tanken om 1000/sek.

![DB-billede6](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/DB-billede6.png)

### 1.5. Software implementation
#### Database
En af de mest essentielle komponenter af systemet var databasen, da det er databasen der skal holde den store mængde data vi skulle arbejde med. Vi valgte at anvende en MySQL database, da der var behov for relationer og en effektiv indexering. Her oprettede vi de nødvendige tabeller, samt deres constraints. Midt i forløbet fik vi specificeret mere præcist, hvilket form dataen skulle have, da Helge gav os specifikationer for det API hans simulering skulle tilgå.

#### Backend
Vores backend blev udviklet ud fra MVC mønstret. Det vil altså sige, at vi oprettede et lag for tilgang til database. Et kontrol lag, som havde at gøre med logikken af koden. Vores løsning blev gradvist ændret i takt med, at mere information blev tilgængeligt, f.eks. hvordan dataen skulle se ud ifølge Helges simulering. Originalt delte vi dataen i henholdsvis Users, Comments & Posts, og forventede, at en posts og kommentarer havde hver deres API entry point til oprettelse. Det viste sig, at både kommentarer og posts kom fra samme endpoint, derfor skelnede mellem dem ud fra deres “post_type”, som et felt i den JSON data vi modtog.

## 2. Maintenance og SLA status
Vi vil i dette afsnit komme ind på hvordan vores Hand-over foregik, hvordan vores SLA blev håndteret og hvordan vi vedligeholdte den gruppes projekt vi skulle operere for.

### 2.1. Hand-over
Hand-over i starten gik først ud på at få noget kommunikation med den gruppe vi skulle være operatører for, dette blev gjort via et program kaldet Discord. Discord er en social og gratis Voice over og IP-applikation, som kan bruges til både at chatte og snakke over. Der blev her lavet en Discord server hvor vi her blev givet et dokument med en beskrivelse af deres API og alle dens endpoints. Der blev også specificeret hvad der skulle observeres og hvordan det skulle observeres. Overleveringen af dette synes vi var tilfredsstillende og forståeligt.

### 2.2. Service-level agreement
Vi troede i starten at man skulle lave en service level agreement til sig selv. Vi fandt på en om at forsiden skulle give svar hurtigere end 100msek, målt fra en en frankfurt droplet til vores deployment. Bagefter fandt vi ud af at det ikke var meningen at man skulle finde på en til sig selv, og at gruppe A skulle give os en. Deres SLA til os gik hovedsageligt ud på at vores forside skulle svare hurtigere end 2-3 sek.
Helge lavede også senere en loadsimulator til forsiden og loggede svartiderne i en graf. Det er os nederst på grafen med de hurtigeste svartider, omtrænt 3msek.

![Stor-Graf](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/Stor%20graf.png)

Til monitoring blev der også lavet et python script som viser brugen af diskplads af databasen, såvel som processorbrug i procent.

![Megabutes](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/Megabytes.png)
![CPU-Usage](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/CPU-usage.png)

### 2.3. Maintenance og reliability
For at gøre det nemmere at holde øje med systemets funktion, og ud fra den undervisning vi har fået, valgte vi at implementere et Monitoring-system. System Monitoring kan sådan set implementeres ved at lægge relevante system-events og metrikker i logfiler, og så manuelt kigge dem igennem med jævne mellemrum. Denne metode er dog problematisk, da den kræver at den menneskelige operatør både husker at checke logfiler, og ikke overser vigtige detaljer. Den giver også et meget begrænset overblik i sig selv, så operatøren skal kende systemet ind og ud for at kunne danne sig et billede af dets funktionsdygtighed.

Derfor er det oplagt at lade computere tage sig af den overordnede databehandling, og gemme relevante metrikker i en Time Series Database, hvorfra de kan undersøges vha. et Query-sprog. Til dette formål har vi valgt Prometheus, der kombinerer et web-scraping baseret Monitoring system, med en indbygget Time Series database, hvor alle valgte værdier bliver gemt og indekseret på tid.
For at Prometheus skal være i stand til at monitorere en webapplikation, skal applikationen servere en plaintext metrics side, der nemt kan genereres med et prometheus instrumenterings-bibliotek, der findes i flere varianter an på hvilket sprog det skal bruges i.
Prometheus understøtter også andre features, bl.a. notifikation via email når en værdi ligger uden for et ønsket område. Vi har dog valgt at lade vores Dashboard løsning tage sig af dette, da den er mere brugervenlig.

Prometheus har indbyggede visualiseringsfunktioner der fungerer, men de er klart rettet mod udviklere og ikke slutbrugere. En af de mest brugte løsninger til at skabe brugervenlige dashboards med monitoreringsvisualiseringer hedder Grafana. Den har indbygget brugerstyring og email alerts, udover en meget kraftig udtræks- og visualiseringsmotor, og dette gør det meget nemt at for os at vedligeholde et Dashboard der skal bruges af slutbrugeren, uden at give dem for meget adgang. Vores dashboard kan ses her:

![Grafana-eksempel](https://github.com/KLMM-LSD/LSD-Exam-Report/blob/master/Resources/Grafana-eksempel.png)

For at understøtte operatør-gruppen i at holde øje med om systemet lever op til vores Service Level Agreement, har vi valgt fem metrics der er relevante: 
- PostID for den seneste post vi har modtaget
- Hvor lang tid det tog Frontend og Databasen at servere den seneste forside-forespørgsel
- Hvorvidt Frontend, Status og Prometheus er ‘oppe’
- Hvor mange Post-forespørgsler vi har modtaget
- Hvor mange HTTP-fejl 500 vi har returneret

Ud fra disse metrics er det muligt at få et rimeligt overblik af systemets sundhed, og en periode der bryder med kravene fra vores SLA vil give klare udslag på graferne.<br />
Grafana har også indbygget funktionalitet til at snakke med email-servere via SMTP, så ved at sende Alerts over en af vores gmail-kontoer, var vi i stand til at opsætte automatiske e-mailnotifikationer når f.eks. forside-latensen overstiger 2-3 sekunder.<br />
Dette Monitoring-system viste sig at være et meget godt værktøj, selv om vi ikke strengt taget fik specielt meget brug for det ift. systemfejl og SLA-brud. Selv når alt kører som det skal, kan det være svært at sikre sig selv om det, og det i sig selv kan meget hurtigt blive til en tidskrævende manuel arbejdsproces, hvis ikke man har opsat de rigtige værktøjer og metoder til at begynde med.

## 3. Diskussion
### 3.1. Teknisk diskussion
Det var et win at lave en Cache til forsiden. Det var et win at lave bulk-indsættelser. Det var et win at udflade databasestrukturen.

Vores forside var i forvejen hurtigst til at loade, på omtrent 3msek. Vi snakkede lidt om gruppe C på et tidspunkt og de sagde at de havde sat en cache for forsiden, som kun opdaterede hver halve time. Det er ikke optimalt kun at opdatere forsiden hver halve time for så er siden ubrugeligt langsom, men det er smart med en cache for det kan forbedre ydelsen. Det meste af tiden der går på at servicere en request går på at generere en JSON-respons. Vi forbedrede vores svartider på forsiden fra 3 msek til 2 msek ved også at lave en forside JSON cache, men vi opdaterer hvis den er ældre end et sekund. Det hjælper også mod spikes i antal requests i sekundet.

### 3.2. Gruppearbejde reflektion & Erfaringer (Lessons learned)
Det endelige produkt virker fint og der var ikke nogle store problemer undervejs. I starten lavede vi det som en stor war-fil, men endte med at lave flere forskellige war-filer der hver havde sit eget formål. Helge ville have at der var et /status endpoint, som skulle kunne rapportere at servicen ikke virker hvis det var tilfældet. Hvis det hele var et stort projekt ville man aldrig kunne rapportere at servicen ikke virker for så ville det hele slet ikke køre. Det var smart for så kunne man opdatere de enkelte komponenter uden at tage resten ud af brug.
 
Helge sagde i en af undervisningstimerne at omtrent 80% af udviklingsomkostningerne af et typisk projekt går på vedligeholdelse. I vores tilfælde var der ingen vedligeholdelse nødvendigt - det virkede bare.<br /> 
Vi holdte systemet forholdsvis simpelt, så der var den funktionalitet der var blevet bedt om og ikke mere. Dette medvirkede til, at vedligeholdelsen af system var gnidningsfrit. Havde vi skulle lægge nye funktioner oven på systemet og gradvist øge kompleksiteten, så kunne det være vi ville mærke nogen af de problemer som hører til vedligeholdelse. Generelt set udviklede vi dog efter designprincipper vedr. simplicitet(nemt at forstå), operabilitet (nemt at anvende) og evolvablity (nemt at udvide).
