---
layout: post
title: "DIY PCB: Vom Design zum fertig bestückten STM32F07-Breakout-Board."
---
Ich habe meine erste Platine entworfen, produzieren lassen, bestückt und programmiert.
Dieser Artikel fasst meine Erfahrungen zusammen.

## Motivation und Mikrocontroller

Ich sitzt viel vor dem Rechner.
Um davon etwas weg zu kommen, habe ich gedacht ich beschäftige mich mit der Herstellung von Platinen mit dem Ziel Löten zu lernen.
Leider habe ich kein vorgefertigtes Board/Set zum Löten gefunden, welches mir gefällt. 
Also muss ein eigenes Board entworfen werden.

Da es mir um den Herstellungsprozess geht, habe ich keine besonderen Anforderungen an einen Mikrocontroller.
Nach einigem stöbern in Online-Katalogen der großen Elektronik-Händler habe ich mich für den [STM32F070CBT6](https://www.st.com/resource/en/datasheet/stm32f070cb.pdf) (PDF) entschieden, weil er eine USB-Schnittstelle hat und relativ günstig ist. 
Außerdem verfügt er über einen vorinstallierten nicht überschreibbaren DFU-Bootloader.
Den Bootloader fand ich wichtig, da dieser garantiert, dass ich den Mikrocontroller nicht mit Software unbrauchbar machen kann. 
Wie ich später gemerkt habe, war diese Sorge unbegründet, da man mittels Serial-Wire-Debug der Mikrocontroller immer programmierbar ist.

Der Schaltplan, das Boardlayout und der Programm-Code des hier vorgestellten Breakout-Boars ist auf [GitHub](https://github.com/der-b/blog-stm32_breakout/) verfügbar.

## Features des Breakout-Boards

Da es mein erstes Breakout-Board ist, habe ich mich nach einer kurzen Recherche, für folgende Features entschieden:

- USB (inkl. Stromversorgung)
- Möglichst viele IO-Port
- Jumper zur Auswahl des Bootloaders
- Eine LED, die anzeigt ob Spannung anliegt
- Zwei programmierbare LEDs
- Zugänglichkeit der Single-Wire-Debug-Schnittstelle.

Dazu kommen noch ein paar zusätzliche Dinge, die zu beachten sind:

- ESD-Schutz für den USB-Port
- Ein Spannungsregler ist nötig, da der Mikrocontroller nur 3,3 Volt verträgt und USB ein 5 Volt Versorgung hat.

<div class="note"> 
	<h5>Hinweis</h5>
	<p>
	Beim recherchieren habe ich nicht gesehen, das ein Quarz an den Mikrocontroller angeschlossen werden muss damit der USB-Port benutzt werden kann.
	Siehe dazu <a href="https://www.st.com/content/ccc/resource/technical/document/application_note/group0/0b/10/63/76/87/7a/47/4b/DM00296349/files/DM00296349.pdf/jcr:content/translations/en.DM00296349.pdf">USB hardware and PCB guidlines using STM32 MCU</a> (PDF) Table 4 in AN4879 Rev 4.
        Das stimmt nur teilweise: 
        Der vorinstallierte DFU-Loader funktioniert tatsächlich nicht, aber mir ist es gelungen eine <a href="#capslock">Beispielanwendung</a> für die USB-Schnittstelle zu implementieren.
	</p>
</div>

# Design Tools

Die zwei wohl bekanntesten Programme zum Entwerfen von Platinen sind wohl [EAGLE](https://en.wikipedia.org/wiki/EAGLE_(program)) und [KiCad](https://en.wikipedia.org/wiki/KiCad).
Da EAGLE kommerzielles Produkt ist, bietet die kostenfreie Version nur eine beschränkten Funktionsumfang.
Auf der Website von KiCad gibt es gute [Tutorials](https://kicad-pcb.org/help/tutorials/).
Ich dokumentiere hier nur, was ich zusätzlich gern gewusst hätte.

# Schaltplan

Trotz der Einfachheit des Boards, gibt es ein paar Dinge zum Schaltplan zu sagen. Den gesamten Schaltplan gibt es [hier](https://github.com/der-b/blog-stm32_breakout/blob/master/schematic/img/schematic.png).

## Der Mikrokontroller

Bei der Bestückung des Mikrocontrollers mit Kapazitäten habe ich mich ans [Datenblatt](https://www.st.com/resource/en/datasheet/stm32f070cb.pdf) gehalten.
Das Kopieren der empfolenen Beschaltung aus Datenblättern ist ein Standardvorgehen.
Das Ergebnis davon ist, dass man meistens viele kleine Kondensatoren hinzufügt, die Spannungsschwankungen ausgleichen.
Aus welchen Grund eine bestimmter Kapazitätswert zur Verfügung gestellt werden muss, bleibt dabei unbegründet.
Allerdings ist es in den meisten Fällen nicht so schlimm, wenn man ein oder zwei Kondensatoren vergisst.

## Spannungsversorgung

Um die 5 Volt der USB-Schnittstelle auf 3,3 Volt zu reduzieren, nutze ich einen Spannungsregler [AZ1117C-3.3](https://www.diodes.com/assets/Datasheets/AZ1117C.pdf) (PDF).
Die Beschaltung des Reglers habe ich auch aus dem Datenblatt entnommen.
 
![Spannungsversorgung](/assets/img/2020/stm32_breakout_spannung.png)
<div class="clear"></div>

## LEDs

Als [Vorwiderstand](https://www.elektronik-kompendium.de/sites/grd/1006011.htm) für die LEDs habe ich einen 200 Ohm Widerstand gewählt.
Ich würde im Nachhinein einen größeren Widerstand wählen, da die LEDs sehr hell leuchten.

## USB

![USB schematic](/assets/img/2020/stm32_breakout_usb.png)
Die USB Beschaltung habe ich entsprechend der [AN4879](https://www.st.com/content/ccc/resource/technical/document/application_note/group0/0b/10/63/76/87/7a/47/4b/DM00296349/files/DM00296349.pdf/jcr:content/translations/en.DM00296349.pdf) (PDF) gestaltet.
Den PullDown-Widerstand zum ID-Pin habe ich im Netz als Tip gefunden.
Ich bin mir aber nicht sicher ob es richtig ist, aber es funktioniert. 

Außerdem habe ich einen ESD-Schutz eingebaut.
Die verwendete ESD-Schutz-Diode ist die [MMBZ10VAL](http://rohmfs.rohm.com/en/products/databook/datasheet/discrete/diode/zener/mmbz10val-e.pdf) (PDF).

## Pin Header

Natürlich habe ich so viel Pins nach Außen gelegt wie möglich.
Allerdings gibt es ein paar Dinge, die man beachten sollte:
Drei Pins habe ich genutzt um eine Spannungsversorgung anschließen zu können. 
Das heißt, einen Pin Ground, einen für 5V und einen für 3.3V.
Dabei ist zu beachten, dass die 5V mittels des Regulators auf 3.3V reduziert.
Der 3.3V Pin ist direkt mit dem Mikrokontroller verbunden.
Da ich keinen Verpolungs-Schutz eingebaut habe, kann eine falsche Beschaltung den Mikrocontroller zerstören.

Da es sich um einen ARM-Mikrocontroller handelt, verfügt auch dieser über eine Single-Wire-Debug-Schnittstelle.
Diese belegt zwei weitere Pins.
Alle weitere Pins habe ich mehr oder minder zufällig herausgeführt.

## Boot Pin

Ich habe auch den Boot Pin hinausgeführt, der mittels eines Jupers auf 3.3 Volt oder auf Ground gezogen wird.
Je nach Beschaltung wird entweder der vorinstallierte DFU-Bootloader gestartet oder das im Flash liegende Programm.
Da der vorinstallierte USB-Boot-Loader nicht ohne einen Quarz funktioniert, muss der Pin immer auf Ground gezogen werden, da ansonsten zwar der DFU-Bootloader gestartet wird, aber nichts macht.

## Footprints

Bevor man jetzt eine PCB-Layout erstellen kann, muss man allen verwendeten Bauteilen einen Footprint zuweisen.
Das kann man in KiCad über die Properties der Bauteile einstellen.
Für die komplexeren Bauteile sind die Footprins eigentlich immer hinterlegt.
Für die einfacheren Bauteile muss man das per Hand einstellen, weil diese meist mit verschiedenen Footprints verfügbar sind.

Widerstände, Kapazitäten und LEDs haben alle den gleichen Footprint, allerdings gibt es die in unterschiedlichen Größen.
Für das Board habe ich mich für den 0603 Footprint entschieden. 
Dabei sind zwei Dinge zu beachten: 1. Sind die Bauteile mit den vorgesehenen Eigenschaften in der Größe verfügbar und 2. sind sie noch ausreichend groß, um sie per Hand zu löten.

Auch der verwendete Regulator wird in verschiedenen Gehäusen angeboten. Ich habe mehrfach überprüft ob der Footprint mit den Teilen, die ich bestellen wollte übereinstimmten. Auch wenn das richtige Gehäuse gewählt wurde, ist darauf zu achten, dass die Pin-Belegung richtig ist.

Das Auswählen der Footprints fand ich recht unangenehm, weil ich dazu kaum Informationen gefunden habe.
Ich habe keine guten Tipps dazu gefunden, wie man am besten vor geht.
[Hier](https://www.electronics-notes.com/articles/electronic_components/surface-mount-technology-smd-smt/packages.php) gibt es eine Übersicht über verschiedene Gehäuse.
Anfangs hatte ich eine SOT-23 anstatt eines SOT-223 für den Regulator ausgewählt.
Die beiden Gehäuse haben nicht mal die gleiche Anzahl an Pins, obwohl die Namen ähnlich klingen.
Wenn es bei den Gehäusen eine Systematik gibt, dann habe ich sie noch nicht verstanden.

# PCB Layout

[![PCB Layout](/assets/img/2020/stm32_breakout_pcb_layout.png)](/assets/img/2020/stm32_breakout_pcb_layout.png)

Ein gutes Vorgehen beim Erstellen eines PCB-Layouts ist es, die größeren Bauteile als erstes zu platzieren.
In diesem Fall sind das die Pin-Header, der Mikrocontroller, der Regulator und der USB-Anschluss.
KiCad hat Möglichkeiten Distanzen zu Layout-Editor zu messen.
Das Feature ist hier goldwert um zum Beispiel die richtigen Abstände zwischen den Pin-Headern zu wählen, so dass das Board auch auf ein Breadboard passt.
Sind die größeren Teile platziert, dann sollte man die äußeren Kanten des Boards festlegen.
Dabei sind schon die Maximalgrößen zu beachten, die vom PCB-Hersteller vorgegeben sind.

Anfangs hatte ich die Größe eines [Arduino Nanos](https://store.arduino.cc/arduino-nano) angepeilt.
Allerdings ist der Mikrocontroller etwas größer als der auf dem Arduino Nano und dadurch hatte ich dann nicht genug Platz um die Pins mit den Controller zu verbinden.
Mein Board ist um einen Pin-Header breiter als der Nano.
Außerdem ist mein Board etwas kürzer, da ich kein Bohrungen für eventuelle Schrauben vorgesehen habe.

Nach der groben Positionierung sollte der Controller ausgerichtet werden.
Das Wichtigste ist es, das Signalleitungen mit hohen Frequenzen möglichst kurz sind.
In diesem Fall sind das die beiden USB-Datenleitungen.
Idealer Weise sind diese auch gleich lang.

KiCad zeichnet jetz ein Spinnennetz aus weißen Linien zwischen den Komponenten an.
Diese Linen zeigen an, welche Kontakte miteinander verbunden werden müssen.
Leider zeigen sie **nicht** die Nähe zwischen den Komponenten im Schematic an!
Das heißt man muss aufpassen, dass die richtigen Kondensatoren auch in der Nähe der richtigen Pins angeordnet sind.

Die Pin-Folge an den Mikrocontroller folgt anscheinend keinem System.
Daher kann es ratsam sein die Pinzuweisung der Pin-Header noch einmal zu ändern um das Routing der Leitungen zu vereinfachen.

Sind alle Komponenten angeordnet und alle Kontakte entsprechend verbunden, dann sollte man immer noch mal ein Update des Layouts vom Schematic durchführen.
Ich muss während der Erstellung des PCB-Layouts versehentlich den Kondensator Nummer 7 gelöscht haben.
Das ist mir erst beim Löten aufgefallen.
Da es nur eine Kapazität war, ist das erstmal nicht weiter schlimm, aber es ist trotzdem ärgerlich und sollte vermieden werden.

Auf den automatische Check der Desigen-Regeln sollte auf keinen Fall verzichtet werden und alle Warnungen behoben werden.

Außerdem solltet die Beschriftung der Boards nicht fehlen.
Besonders wichtig ist es die Spannungsversorgung korrekt zu markieren. 
Fehlt diese, muss im schlimmsten Fall jedes mal KiCad geöffnet werden um zu sehen, welcher Pin für Ground und welcher für 5V war.
Das nervt sehr schnell.
Dabei sollte man bedenken, dass das Board am Rechner deutlich größer dargestellt wird als es in Realität ist.
Ich habe die Standard-Schriftgröße verkleinert, weil ich nicht genug Platz hatte und das führte dazu, dass die Beschriftung auf dem Board nur noch schwer lesbar ist.

Am Ende müssen die Dateien für den Board-Hersteller generiert werden.
Dazu folgt man am besten den Anweisungen von der Webseite des Herstellers.

# PCB-Hersteller

Zur Auswahl eines Herstellers kann ich keine guten Tipps geben.
Das Paket meines ersten Herstellers ist nie angekommen.

Als zweiten Hersteller habe ich [JLCPCB](https://jlcpcb.com/) ausprobiert und das lief sehr gut.
Allerdings ist mein Urteil hier mit Vorsicht zu genießen, weil ich nicht viel Erfahrung mit anderen Herstellern habe.

Wenn man den günstigsten Versand auswählt, dann muss man nach der Bestellung normalerweise zwei bis drei Wochen warten bis die Platinen ankommen.
Bei chinesischen Herstellern ist zu beachten, dass im ganzen Land zwei Mal im Jahr so gut wie niemand arbeitet.
Das ist einmal zum [Chinesischen Neujahr](https://de.wikipedia.org/wiki/Chinesisches_Neujahrsfest) und einmal zum [Nationalfeiertag in China](https://en.wikipedia.org/wiki/National_Day_of_the_People%27s_Republic_of_China).
Bestellungen, die in diesem Zeitraum getätigt werden, haben längere Lieferzeiten unabhängig vom gewählten Versand.

# Komponenten

Es gibt einige Lieferanten für die Komponenten.
Bevor man alle Einzelteile sucht und in den Einkaufswagen legt, sollte man sicherstellen, dass der gewählte Lieferand auch an Privatpersonen liefert.
Mein zu erst gewählter Lieferand verkauft nicht an Privatpersonen und hat daher direkt die Bestellung storniert.

Man sollte mehr Komponenten bestellen, als man braucht, weil sie in gößeren Mengen pro Stück günstiger werden und wenn man ein paar beschädigt, dann hat man Ersatz.
Alles was man zu viel hat, kann man für spätere Projekte wiederverwenden.

Ich habe bei [DigiKey](https://www.digikey.de/) bestellt und habe keine Beanstandungen.
Allerdings gilt auch hier, dass ich unerfahren bin und daher nicht beurteilen kann, was gut oder nicht gut ist.

# Zusammenbau

Als die Komponenten und auch die Platinen angekommen waren, war ich von der Größe überrascht.
Das alles ist viel kleiner, als ich es erwartet hatte.
Das folgende Bild zeigt das fertig zusammengesetzte Board mit meinen Zeigefinger als Größerenreferenz:

[![Board](/assets/img/2020/stm32_breakout_size_small.jpg)](/assets/img/2020/stm32_breakout_size.jpg)

Als Erstes stellt sich die Frage, welche Teile man als erste auflötet.
Da ich das vorher noch nicht gemacht hatte, habe ich mit den einfachen Teilen angefangen.
Davon rate ich ab!
Ich hatte ein Board fast komplett besetzt, aber habe dann den USB-Anschluss nicht richtig gesetzt bekommen und dadurch das ganze Board versaut.
Besser finde ich es mit den schwierigen Teilen anzufangen, also den USB-Anschluss und den Mikrocontroller.

Persönlich fand ich den USB-Anschluss am schwersten, da das Gehäuse die Pins teilweise verdekt und sobald man mit dem Lötkolben ansetzt nicht mehr sehen kann, was man tut. Hier einer meiner ersten Vesuche:

![USB Anschluss](/assets/img/2020/stm32_breakout_usb1.jpg)

Die Datenleitungen sind teilweise überbrückt und das Gehäuse ist mit Lötzinn überzogen.
Die beste Methode war es, als Erstes mit einem einzelnen Pin anzufangen.
Dabei habe ich auf einen Pin-Pad der Platine etwas Lötzinn aufgebracht und anschließend den USB-Anschluss entsprechend ausgerichtet um dann den Pin zu verlöten.
Anschließend habe ich die anderen Pins befestigt. 
Bevor ich das Gehäuse befestigte, habe ich die Kontakte überprüft: Man steckt vorsichtig ein USB-Kabel ein und stellt dann mittels einer Durchgangsprüfung sicher, dass die Pins nicht miteinander verbunden sind.
Anschließend entfernt man vorsichtig das USB-Kabel wieder und befestigt das Gehäuse.

Als Zweites habe ich den Mikrocontroller aufgelötet.
Es gibt unzählige Videos, die zeigen wie man einen Mikrocontroller am besten anlötetet (z.B.: [Mikrocontroller Löten](https://www.youtube.com/watch?v=QiIy3Oe2UGI)).
Ich habe im Prinzip eine von diesen Methoden verwendet.
Beim Ausrichten des Controllers ist sehr viel Geduld gefragt.
Das sieht in den Videos einfacher aus, als es ist.
Hier einer meiner Versuche, bei dem die Ausrichtung fehlgeschlagen ist und die Pins einen Kurzschluss zwischen den Pads erzeugen:

![Versaute Pins 1](/assets/img/2020/stm32_breakout_uc_pin.jpg)

Selbst bei meinem fertigen Board sind die Pins auf einer Seite schlecht ausgerichtet, wie im nächsten Bild zu sehen ist.
Alle anderen Seiten sehen einigermaßen gut aus.
Trotzdessen funktioniert alles.

[![Versaute Pins 2](/assets/img/2020/stm32_breakout_uc_pin2_small.jpg)](/assets/img/2020/stm32_breakout_uc_pin2.jpg)

In den oben genannten Viedos zum Löten wird auch Entlötlitze verwendet.
Leider erwähnen sie in den Videos nicht, dass man bei der Nutzung der Entlötlitze manchmal Pins volständig von der Platine ablötet und man eine kalte Lötstelle erzeugt.
Ich habe mehrmals kalte Lötstellen nacharbeiten müssen. 
Besonders deprimierend kann es werden, wenn man das Board das erste Mal versucht in Betrieb zu nehmen und es nicht funktioniert.
Es ist durchaus möglich, dass es nur eine kalte Lötstelle ist.

Wärend bei den Widerständen und den Kapazitäten nicht auf die Polung geachtet werden muss, ist das bei den LEDs notwendig und garnicht so einfach. 
Mein LEDs haben auf der rückseite eine Markierung für die Stromrichtung (links ist mein Daumen zu sehen): 

[![LED von hinten](/assets/img/2020/LED_back_small.jpg)](/assets/img/2020/LED_back.jpg)

Leider sind die LEDs so klein, dass sie sich nur schwer umdrehen lassen und wenn sie denn umgedreht sind, weiß man nicht sicher, ob ich sie aus versehen auch um eine andere Achse gedreht habe. 

![Durchgangsprüfung](/assets/img/2020/LED_on_off.gif)

Ich habe das Problem gelöst, in dem ich mit dem Multimeter eine Durchgangsprüfung durchgeführt habe.
Legt man die Kontakte in die richtige Richtung an, dann fäng die LED zu glimmen an, wie man es im GIF sehen kann.
<div class="clear"></div>

# Inbetriebnahme

Wenn man das erste Mal eine selbst bestückte Platine in Betieb nimmt, sollte man vorsichtig sein, weil man eventuell Kurzschlüsse hat.
Dies kann dazu führen, dass der Mikrocontroller anfängt zu rauchen, daher nennt man die erste Inbetriebnahme auch [Smoke-Test](https://en.wikipedia.org/wiki/Smoke_testing_(electrical)).
Dazu sollte man **nicht** die USB-Buchse seines eigenen Laptop benutzen.
Eigentlich macht man das mit einem Labor-Netzteil, aber wenn man gerade das erste Mal ein Board entwickelt hat, dann hat man höchst wahrscheinlich auch kein Labor-Netzteil zur Verfügung.
Ich habe einen  billigen 5 Volt Adapter dafür genutzt.

Hat das Board den Smoke-Test überstannden, dann gilt es zu überprüfen, ob der Mikrocontroller auch ein Lebenszeichen von sich gibt.
Ich hatte gehoft, dass ich die eingebrannte Firmware des [STM32F070CBT6](https://www.st.com/resource/en/datasheet/stm32f070cb.pdf) (PDF) nutzen kann um meine Programme hoch zu laden.
Leider meldet sich der Controller nicht als USB-Device, wenn er an eine Rechner angesteckt wird.
Für den Notfall hatte ich im Vorhinein schon einen günstigen Programmer besorgt, der mittels der Single-Wire-Debug-Schnittstelle den Mikrocontroller programieren kann.

![ST-Link_v2_clone](/assets/img/2020/ST_Link_V2.jpg)

Damit man den Programmer benutzen kann, braucht man die [stlink-tools](https://github.com/texane/stlink).
Unter Arch-Linux installiert man die Tools mittels:

```
$ pacman -S stlink
```

Damit werden mehrer Programme installiert.
Das vorerst interessanteste ist *st-info* welche man nutzen kann um den angeschlossenen Mikrocontroller-Typ zu ermitteln.
Leide bekam ich folgende Ausgabe:

```
$ st-info --probe
Found 1 stlink programmers
 serial: 303030303030303030303031
openocd: "\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x31"
  flash: 0 (pagesize: 0)
   sram: 0
 chipid: 0x0000
  descr: unknown device
```

Das sah so aus, als ob mein Board nicht funktionierte oder ich den Mikrocontroller beschädigt hätte.
Nach einigen Experimenten und einigen Untersuchungen mit dem Oszilloskop konnte ich den Fehler aus machen.
Ich hatte den Mikrokontroller in einen Breadboard gesteckt und das Kable welches die SWIO-Verbindung zu dem Programmer herstellen sollte hatte einen Wackelkontakt.
Anschließend bekam ich die gewünschte Ausgabe:

```
$ st-info --probe
Found 1 stlink programmers
 serial: 303030303030303030303031
openocd: "\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x30\x31"
  flash: 131072 (pagesize: 2048)
   sram: 16384
 chipid: 0x0448
  descr: F07x device
```

# Software

Natürlich möchte man als nächstes den Mikrocontroller programmieren.
Da ich einen [Tomu](http://tomu.im/tomu.html) besitze habe ich bereits ein wenig Erfahrung gehabt in der Programmierung von ARM-Mikrokontrollern.
Daher haben meine Code-Beispiele auch eine Ähnlichkeit mit den [Beispiel-Programmen](https://github.com/im-tomu/tomu-quickstart) für den Tomu.

Die beiden vorgestellten Programme ([Blink](https://github.com/der-b/blog-stm32_breakout/tree/master/software/blink) und [Capslock](https://github.com/der-b/blog-stm32_breakout/tree/master/software/test-capslock)) sind auf GitHub verfügbar.

## Vorraussetzungen

Um den Mikrocontroller programmieren zu könnnen braucht man natürlich einen Compiler, der die entsprechenden ARM-Binaries erstellt. 
Ich nutze den *gcc* der auch ARM Mikrocontroller unterstützt.
Allerdings muss man normalerweise die ARM Erweiterungen installieren.
Unter Arch Linux installiert man sie mittels:

``` bash
$ pacman -S arm-none-eabi-gcc
```
Theoretisch kann man jetzt mit dem [Datenblatt](https://www.st.com/resource/en/datasheet/stm32f070cb.pdf) (PDF) und dem [Reference Manual](https://www.st.com/resource/en/datasheet/stm32f070cb.pdf) (PDF) anfangen zu programmieren.
Allerdings müsste man dann auch alle Registeraddressen selber aus den Manual auflösen.
Um mir das zu sparen nutze ich [libopencm3](https://github.com/libopencm3).
Der Source Code ist in den Unterordnern der Programme abgelegt.

Zum kompilieren der Programme führt man folgende Befehle aus:

``` bash
$ TARGETS=stm32/f0 make -C libopencm3
$ make
```

Um sie auf den Boards zu laden führt man folgenden Befehl aus:

``` bash
$ make download
```

Allerdings müssen dazu die [stlink-tools](https://github.com/texane/stlink) installiert sein.

## Blink

Das erste Programm ist das "Hello World!"-Programm der Mikrocontroller, welches einfach die LEDs blinken lässt.
Dieses [Programm](https://github.com/der-b/blog-stm32_breakout/blob/master/software/blink/main.c) lässte die beiden LEDs abwechselnd blinken und ist recht einfach.

``` c
#include <libopencm3/stm32/rcc.h>
#include <libopencm3/stm32/gpio.h>

#define LED1_PORT GPIOA
#define LED1_PIN  GPIO4

#define LED0_PORT GPIOA
#define LED0_PIN  GPIO3

int main(void)
{
        // enable the GPIO port A
        rcc_periph_clock_enable(RCC_GPIOA);

        // set LED pins as floating output with pull up resistor
        gpio_mode_setup(LED0_PORT, GPIO_MODE_OUTPUT, GPIO_PUPD_PULLUP, LED0_PIN);
        gpio_mode_setup(LED1_PORT, GPIO_MODE_OUTPUT, GPIO_PUPD_PULLUP, LED1_PIN);

        // turn on LED0
        gpio_set(LED0_PORT, LED0_PIN);

        // turn off LED1
        gpio_clear(LED1_PORT, LED1_PIN);

        while(1) {
                // busy wait some time
                for (int i = 0; i < 500000; ++i) {
                        __asm__("nop");
                }

                // change the state of both LEDs
                gpio_toggle(LED1_PORT, LED0_PIN | LED1_PIN);
        }

        return 0;
}
```

Allerdings wollte ich erst nur die LED0 blinken lassen, was nicht funktionierte.
Ich hatte schon Angst, dass ich den ganzen GPIO-Port A wärend des Zusammenbaus beschädigt hatte, aber es war wieder nur eine kalte Lötstelle.

## Capslock

Das Programm [Capslock](https://github.com/der-b/blog-stm32_breakout/tree/master/software/test-capslock) registriert den Mikrocontroller als USB Tastatur, wenn er mit einem Rechner verbunden wird.
Wird Caps Lock eingeschaltet, dann wird die LED1 eingeschaltet und wenn Num Lock eingeschaltet wird, dann wird LED0 eingeschaltet.
Dieses Program basiert auf einem [Programm](https://github.com/der-b/tomu-capslock), welches ich in der Vergangenheit für den [Tomu](https://tomu.im/) geschrieben habe.

Nach den Informationen der Application Note [AN4879](https://www.st.com/content/ccc/resource/technical/document/application_note/group0/0b/20/63/76/87/7a/47/4b/DM00296349/files/DM00296349.pdf/jcr:content/translations/en.DM00296349.pdf) (PDF) in Table 4 unterstützt der Mikrocontroller kein USB ohne einen externen Quarz anzuschließen.
Tatsächlich funktioniert die vorinstallierte Firmware nicht über USB, wenn kein Quarz angeschlossen ist, aber man kann eigene Programme schreiben, die auch ohne Quarz funktionieren.
Dazu muss man als erstes die Taktfrequenz auf 48Mhz stellen.
Anschließend muss die USB Clock Source auf den PLL gestellt werden im Register RCC\_CFGR3.
Per default ist die USB Clock Source abgeschaltet.
Dann muss die USB Peripheral Clock eingeschaltet werden im Register RCC\_APB1ENR und danach kann man die USB Erweiterung des Mikrocontrollers benutzen.
Es hat einige Zeit gedauert bis ich das mit dem RCC\_CFGR3 register herausgefunden habe.

Mit [libopencm3](https://github.com/libopencm3/libopencm3) sieht das ganze so aus:

``` c
// set processor clock to 48mhz
rcc_clock_setup_in_hsi_out_48mhz();
// select the USB clock source
rcc_set_usbclk_source(RCC_PLL);
// enable the usb peripheral clock
rcc_periph_clock_enable(RCC_USB);
// now the usb extension can be used
usbd_dev = usbd_init(...);
```

Ich vermute, dass die interne Clock einen zu hohen Drift hat um garantieren zu können, dass USB unter allen Bedingungen funktioniert.
Und deshalb steht in der Application Note [AN4879](https://www.st.com/content/ccc/resource/technical/document/application_note/group0/0b/20/63/76/87/7a/47/4b/DM00296349/files/DM00296349.pdf/jcr:content/translations/en.DM00296349.pdf) (PDF), dass USB nur mit angeschlossenem Quarz funktioniert.

# Zusammenfassung

Für mein erstes selbst entworfenes Board bin ich sehr zufrieden.
Ich bin erstaunt, wie einfach das ganze ist.
Alledings kann ich im Nachhinein nicht sagen wie lange ich mich vorher damit beschäftigt habe, wie so ein Entwurfs-Prozess aussehen kann.
Wenn man sich schon mit dem Arduino beschäftigt hat, dann ist der Schritt zum eigenen Board mit moderatem Aufwand auch machbar.

Mein Sekundarziel, weniger vor dem Rechner zu sitzen und mehr mit in der physischen Welt zu sein, betrachte ich als gescheitert.
Die meiste Zeit verbringt man dann doch mit Recherche, Design und Programmieren.
Das eigentliche Löten beansprucht relativ wenig Zeit.
