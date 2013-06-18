---
title: HbbTV-Tutorial 1. Teil
date: 2013-06-17
tags: deutsch, hbbtv
---

# Der Weg zur ersten HbbTV-App Schritt für Schritt

Der Einstieg in eine neues Thema ist häufig etwas schwierig, da man anfangs nicht weiss womit man beginnen und auf was man achten soll. Diese Einführung soll den Einstieg erleichtern. Ich versuche mich dabei am Anfang auf das Wesentliche einer HbbTV-App zu beschränken, so dass wir das nötige Know-How haben um eine erste App zu entwickeln. In den folgenden Posts werden wir dann in die Tiefe gehen und und uns mit den Details der HbbTV-App-Entwicklung beschäftigen.

READMORE

## Warm up

HbbTV steht für Hybrid Broadcast Broadband TV, also eine Mischung aus Rundfunk- und Internet-Technik. Für den Enwickler der HbbTV-App steht der Internet-Teil dieses Standards im Vordergrund, denn eine solche App ist in aller erster Linie eine normale auf XHTML basierte Webseite. Der Runfunktechnik-Teil dieses Standards beschränkt sich weitestgehends auf die Signalisierung der Anwendung im Sendesignal und den Zugriff über eine JavaScript-API auf Informationen des Sendesignals und des Fernsehers und dessen Peripherie wie z.B. die Fernbedienung.

Der aktuelle Standard hat die Version 1.5 und wurde im November 2012 verabschiedet und die ersten Geräte kommen gerade auf den Markt. Die wichtigste Neuerung dieser Version ist die Unterstützung von Adaptive-Streaming nach dem MPEG DASH Standard und eine erweiterter Zugriff auf die EPG-Daten im Sendesignal. Aktuell wird am 2.0er-Standard gearbeitet. Die Verbreitung der 1.5er Geräte ist aber aktuell noch so gering, das wir uns vorerst auf die Version 1.0 beschränken werden. Übrigens meldet der User-Agent des Browser die Version 1.1.1 für HbbTV 1.0 und 1.2.1 wenn er die Version 1.5 meint.

Bevor wir mit dem Programmieren anfangen können brauchen wir noch das nötige Toolkit um die Seiten testen zu können. Dafür eignet sich das [Fire-HbbTV-Plugin](http://tum-iptv.aw.atosorigin.com/firehbbtv/index.php) für Firefox und der [Opera TV Emulator](http://business.opera.com/solutions/tv/emulator) der unter VirtualBox läuft.

Das Firefox-Plugin ist zwar praktisch, aber es gibt keine Garantie, dass die App dann auch auf den Geräten fehlerfrei läuft, denn die Gerätehersteller verwenden selber meistens Opera oder einen Webkit-Browser. Der Browser von Mozilla ist hier nicht vertreten. Die Geräte von Samsung aus 2011 haben eine eigene Browser-Implementierung namens Maple und damit noch einen relevanten Marktanteil. Relativ selten und vornehmlich auf Set-Top-Boxen (STB) vertreten sind inzwischen die embeded Browser von [ANT](http://www.antlimited.com/galio_hbbtv.asp) oder [NetFront](http://gl.access-company.com/products/itelectoronics/hbbtv/) von Access.

Bevor man nun mit der Entwicklung seiner ersten App beginnen kann, ist noch eine Hürde zu nehmen. Wer nicht einen Fernsehsender sein Eigen nennt tut sich schwer ohne das nötige Equipment ein Sendesignal mit einer gültigen HbbTV-AIT zu generieren. Hier ist es einfacher das lokale Netzwerk zu manipulieren und einem existierenden Sender seine selbst programmierte Seite unterzuschieben. Man kann das über einen Debugging-Tool wie den [Charles-Proxy](http://www.charlesproxy.com) tun oder kostengünstiger mit einem eigenen Name-Server, der die Domain einer der Fernsehsender auflöst. Die zweite Methode ist sogar sicherer, da man nicht bei allen Geräten einen Proxy konfigurieren kann.

## Hello HbbTV World

Mit dem nötigen Equipment ausgestattet können wir uns jetzt an die Programmierung der HbbTV-App wagen. Sie ist  letztlich nur eine Webseite, die sich über den Fernsehbildschirm legt. Schaltet man bestimmte Bereiche dieser Seite transparent so kann man dort das Fernsehsignal im Hintergrund sehen auch die Opazität lässt sich über CSS regeln, so dass die HTML-Seite halbtransparent erscheinen kann. Diese Technik wird auch auf der HbbTV-Startseite der Sender verwendet um so genannte Red-Button-Einblendung anzuzeigen. Die URL dieser Seite wird im Sendesignal mit geschickt und ein HbbTV-fähiges Gerät lädt diese Seite sobald der Zuschauer auf diesen Sender wechselt. Diese Seite ist in aller Regel komplett transparent und enthält ein kleines halbtransparentes Element in der rechten unteren Ecke. Der Zuschauer muss also gar nichts tun um HbbTV zu starten, es startet sich von selbst. Nach ein paar Sekunden wird dieses Element mit display:none unsichtbar gemacht und die App wartet dann bis der Zuschauer auf den roten Knopf der Fernbedienung drückt.

Um jetzt eine solche Seite programmieren zu können müssen wir eigentlich nur zwei Dinge wissen: Wie groß ist der Bildschirm und wie kann die App den Knopfdruck auf der Fernbedienung erkennen. Der Bildschirm ist immer 1280x720 Pixel groß, egal ob der Fernseher HD oder Full HD, also 1920x1080 unterstützt. Die App muss also nicht an verschiedene Fenstergrößen angepasst werden. Dennoch ist dieses Thema damit noch nicht beendet, denn nicht alle Geräte stellen alle diese 1280x720 Bildpunkte auf dem Fernseher dar und unterschlagen an den Rändern einen mehr oder weniger breiten Streifen. Besonders Set-Top-Boxen (STB) die an analoge Fernseher angeschlossen sind tun das ziemlich gerne. Dafür wurde im HbbTV-Standard die Safe-Area (1024x684) definiert, also ein Bildschirmbereich der auf dem Fernsehgerät angezeigt werden muss. Ein gutes Layout zeichnet sich nun dadurch aus, dass alle für den Zuschauer relevanten Informationen in der Save-Area zu sehen sind und die Seite auf allen Geräten ausgewogen ausschaut. Eine Abfrage wie groß dieser sichtbare Bereich ist gibt es leider nicht.

![Save area](savearea.jpg) 

## Remote Control

Und nun zur Fernbedienung: Die Abfrage der Button-Events läuft wie bei JavaScript üblich über Event-Listener. Um jetzt die Events den Tasten auf der Fernbedienung zuzuordnen gibt es für jede Taste auf der Fernbedienung einen Key-Code (Event.keyCode). Da die Hersteller bei der Implementierung leider etwas uneinheitlich waren ist es ratsam die keycode.js (des IRTs?) zu verwenden.

Das Bemerkenswerte an den Event-Listenern ist das damit alle Tasten der Fernbedienung von der App abgefangen und neu belegt werden können und das im Prinzip sobald der Zuschauer auf den Sender gewechselt ist und die Startseite geladen ist. Die Ausnahme sind hier die Tasten um im Programm vor und zurück zu springen, die Ein-Austaste und Tasten die für die jeweiligen Gerätehersteller spezifisch sind. Um dem entgegen zu wirken schreibt HbbTV-Standard dem App-Entwickler vor, die Startseite nach ein paar Sekunden auszublenden und die Tastatur-Events für das Fernsehgerät freizugeben. In diesem Status darf die App nur auf den sogenannten Red Button reagieren. Es ist ebenfalls vorgeschrieben, dass beim Druck auf die rote Taste die Anwendung in den ausgeblendeten Zustand übergehen muss und die Tastatur wie oben beschrieben wieder freigeben soll. Die grüne, gelbe und blaue Taste dürfen frei belegt werden.

Der Zugriff auf die HbbTV-API läuft üblicherweise über 

    <div style="visibility: hidden; width: 0; height: 0;">
    <object type="application/oipfApplicationManager" id="oipfAppMan" />
    <object type="application/oipfConfiguration" id="oipfConfig" />
    </div>

