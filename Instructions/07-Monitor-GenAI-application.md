---
lab:
  title: Überwachen der Anwendung für generative KI
  description: 'Hier erfahren Sie, wie Sie Interaktionen mit Ihrem bereitgestellten Modell überwachen und Erkenntnisse zur Optimierung ihrer Nutzung mit der Anwendung für generative KI erhalten.'
---

# Überwachen der Anwendung für generative KI

Diese Übung dauert ca. **30** Minuten.

> **Hinweis**: Diese Übung setzt gewisse Kenntnisse von Azure KI Foundry voraus. Aus diesem Grund sind einige Anweisungen bewusst weniger detailliert gehalten, um eine aktivere Erkundung und praktisches Lernen zu fördern.

## Einführung

In dieser Übung aktivieren Sie die Überwachung für eine App zum Abschließen von Chats und zeigen deren Leistung in Azure Monitor an. Sie interagieren mit Ihrem bereitgestellten Modell, um Daten zu generieren, die generierten Daten über das Dashboard „Einblicke für generative KI-Anwendungen“ anzuzeigen und Warnmeldungen einzurichten, um die Bereitstellung des Modells zu optimieren.

## Einrichten der Umgebung

Um diese Übung abzuschließen benötigen Sie Folgendes:

- Ein Azure KI Foundry-Projekt
- Ein bereitgestelltes Modell (z. B. GPT-4o),
- Eine verbundene Application Insights-Ressource.

### Bereitstellen Sie ein DALL-E-Modell in einem Azure KI Foundry-Projekt

Um schnell ein Azure AI Foundry-Projekt einzurichten, finden Sie unten einfache Anweisungen zur Verwendung der Benutzeroberfläche des Azure AI Foundry-Portals.

1. Öffnen Sie in einem Webbrowser unter `https://ai.azure.com` das [Azure KI Foundry-Portal](https://ai.azure.com) und melden Sie sich mit Ihren Azure-Anmeldeinformationen an.
1. Suchen Sie auf der Startseite im Abschnitt **Modelle und Funktionen erkunden** nach dem Modell `gpt-4o`, das wir in unserem Projekt verwenden werden.
1. Wählen Sie in den Suchergebnissen das Modell **gpt-4o** aus, um dessen Details anzuzeigen, und wählen Sie dann oben auf der Seite für das Modell die Option **Dieses Modell verwenden** aus.
1. Wenn Sie zum Erstellen eines Projekts aufgefordert werden, geben Sie einen gültigen Namen für Ihr Projekt ein und erweitern Sie **Erweiterte Optionen**.
1. Wählen Sie **Anpassen** aus und legen Sie die folgenden Einstellungen für Ihr Projekt fest:
    - **Azure KI Foundry-Ressource**: *Ein gültiger Name für Ihre Azure KI Foundry-Ressource*
    - **Abonnement:** *Geben Sie Ihr Azure-Abonnement an.*
    - **Ressourcengruppe**: *Erstellen Sie eine Ressourcengruppe, oder wählen Sie eine Ressourcengruppe aus*.
    - **Region**: *Wählen Sie einen beliebigen Standort aus, an dem KI Services unterstützt wird***\*

    > \* Einige Azure KI-Ressourcen unterliegen regionalen Modellkontingenten. Sollte im weiteren Verlauf der Übung eine Kontingentgrenze überschritten werden, müssen Sie möglicherweise eine weitere Ressource in einer anderen Region anlegen.

1. Wählen Sie **Erstellen** und warten Sie, bis Ihr Projekt einschließlich der von Ihnen ausgewählten GPT-4-Modellbereitstellung erstellt wurde.
1. Wählen Sie im Navigationsbereich auf der linken Seite **Übersicht** aus, um die Hauptseite für Ihr Projekt anzuzeigen.
1. Stellen Sie im Bereich **Endpunkte und Schlüssel** sicher, dass die **Azure AI Foundry**-Bibliothek ausgewählt ist, und zeigen Sie den **Azure AI Foundry-Projektendpunkt** an.
1. **Speichern** Sie den Endpunkt in einem Editor. Sie verwenden diesen Endpunkt, um in einer Clientanwendung eine Verbindung zu Ihrem Projekt herzustellen.

### Verbinden von Application Insights

Verbinden Sie Application Insights mit Ihrem Projekt in Azure AI Foundry, um mit dem Sammeln von Daten für die Überwachung zu beginnen.

1. Verwenden Sie das Menü auf der linken Seite, und wählen Sie die Seite **Ablaufverfolgung** aus.
1. **Erstellen Sie eine neue** Application Insights-Ressource, um eine Verbindung mit Ihrer App herzustellen.
1. Geben Sie einen Application Insights-Ressourcennamen ein, und wählen Sie **Erstellen** aus.

Application Insights ist jetzt mit Ihrem Projekt verbunden, und die Daten werden für die Analyse erfasst.

## Interagieren mit einem bereitgestellten Modell

Sie interagieren programmgesteuert mit Ihrem bereitgestellten Modell, indem Sie eine Verbindung mit Ihrem Azure KI Foundry-Projekt mithilfe von Azure Cloud Shell einrichten. Auf diese Weise können Sie eine Eingabeaufforderung an das Modell senden und Überwachungsdaten generieren.

### Herstellen einer Verbindung mit einem Modell über die Cloud Shell

Beginnen Sie, indem Sie die erforderlichen Informationen abrufen, die für die Interaktion mit Ihrem Modell authentifiziert werden sollen. Anschließend greifen Sie auf die Azure Cloud Shell zu und aktualisieren die Konfiguration, um die bereitgestellten Eingabeaufforderungen an Ihr eigenes bereitgestelltes Modell zu senden.

1. Öffnen Sie eine neue Browserregisterkarte (wobei das Azure AI Foundry-Portal auf der vorhandenen Registerkarte geöffnet bleibt).
1. Öffnen Sie in dem neuen Tab das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` und melden Sie sich gegebenenfalls mit Ihren Azure-Anmeldeinformationen an.
1. Verwenden Sie die Schaltfläche **[\>_]** rechts neben der Suchleiste oben auf der Seite, um eine neue Cloud-Shell im Azure-Portal zu erstellen, und wählen Sie eine ***PowerShell***-Umgebung ohne Speicher in Ihrem Abonnement.
1. Wählen Sie in der Cloud Shell-Symbolleiste im Menü **Einstellungen** die Option **Zur klassischen Version wechseln**.

    **<font color="red">Stellen Sie sicher, dass Sie zur klassischen Version der Cloud Shell gewechselt haben, bevor Sie fortfahren.</font>**

1. Geben Sie im Cloud Shell-Bereich die folgenden Befehle ein und führen Sie sie aus:

    ```
    rm -r mslearn-genaiops -f
    git clone https://github.com/microsoftlearning/mslearn-genaiops mslearn-genaiops
    ```

    Mit diesem Befehl wird das GitHub-Repository geklont, das die Codedateien für diese Übung enthält.

    > **TIPP**: Wenn Sie Befehle in die Cloudshell einfügen, kann die Ausgabe einen großen Teil des Bildschirmpuffers in Anspruch nehmen. Sie können den Bildschirm löschen, indem Sie den Befehl `cls` eingeben, um sich besser auf die einzelnen Aufgaben konzentrieren zu können.

1. Navigieren Sie nach dem Klonen des Repositorys zu dem Ordner, der die Codedateien der Anwendung enthält:  

    ```
   cd mslearn-genaiops/Files/07
    ```

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv openai azure-identity azure-ai-projects azure-ai-inference azure-monitor-opentelemetry
    ```

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu öffnen:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. In der Code-Datei:

    1. Ersetzen Sie in der Code-Datei den Platzhalter **your_project_endpoint** durch den Endpunkt für Ihr Projekt (kopiert von der Projektseite **Übersicht** im Azure AI Foundry-Portal).
    1. Ersetzen Sie den Platzhalter **your_model_deployment** durch den Namen, den Sie Ihrer GPT-4o-Modellbereitstellung zugewiesen haben (standardmäßig `gpt-4o`).

1. *Nachdem* Sie die Platzhalter ersetzt haben, verwenden Sie im Code-Editor den Befehl **STRG+S**, oder **klicken Sie mit der rechten Maustaste und klicken dann auf „Speichern“**, um **Ihre Änderungen zu speichern**. Verwenden Sie dann den Befehl **STRG+Q**, oder **klicken Sie mit der rechten Maustaste und klicken dann auf „Beenden“**, um den Code-Editor zu schließen, während die Cloud Shell-Befehlszeile geöffnet bleibt.

### Senden von Eingabeaufforderungen an Ihr bereitgestelltes Modell

Sie führen nun mehrere Skripts aus, die verschiedene Prompts an Ihre bereitgestellten Modelle senden. Diese Interaktionen generieren Daten, die Sie später in Azure Monitor beobachten können.

1. Führen Sie den folgenden Befehl aus, um **das erste Skript** anzuzeigen, das bereitgestellt wurde:

    ```
   code start-prompt.py
    ```

1. Geben Sie im Befehlszeilenbereich der Cloud-Shell den folgenden Befehl ein, um sich bei Azure anzumelden.

    ```
   az login
    ```

    **<font color="red">Sie müssen sich bei Azure anmelden - auch wenn die Cloud-Shell-Sitzung bereits authentifiziert ist.</font>**

    > **Hinweis**: In den meisten Szenarien ist nur die Verwendung von *az login* ausreichend. Wenn Sie jedoch Abonnements in mehreren Mandqanten haben, müssen Sie möglicherweise den Mandanten mit dem Parameter *--tenant* angeben. Weitere Informationen finden Sie unter [Interaktive Anmeldung bei Azure mit der Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Wenn Sie dazu aufgefordert werden, folgen Sie den Anweisungen, um die Anmeldeseite in einer neuen Registerkarte zu öffnen, und geben Sie den angegebenen Authentifizierungscode und Ihre Azure-Anmeldeinformationen ein. Schließen Sie dann den Anmeldevorgang in der Befehlszeile ab, und wählen Sie das Abonnement aus, das Ihren Azure AI Foundry Hub enthält, wenn Sie dazu aufgefordert werden.
1. Geben Sie nach der Anmeldung den folgenden Befehl ein, um die Anwendung auszuführen:

    ```
   python start-prompt.py
    ```

    Das Modell generiert eine Antwort, die mit Application Insights für eine weitere Analyse erfasst wird. Lassen Sie uns unsere Eingabeaufforderungen variieren, um ihre Effekte zu untersuchen.

1. **Öffnen und überprüfen Sie das Skript**, wo der Prompt das Modell anweist, **nur mit einem Satz und einer Liste zu antworten**:

    ```
   code short-prompt.py
    ```

1. **Führen Sie das Skript aus** , indem Sie den folgenden Befehl in die Befehlszeile eingeben:

    ```
   python short-prompt.py
    ```

1. Das nächste Skript hat ein ähnliches Ziel, enthält aber die Anweisungen für die Ausgabe in der **Systemnachricht** anstelle der Benutzernachricht:

    ```
   code system-prompt.py
    ```

1. **Führen Sie das Skript aus** , indem Sie den folgenden Befehl in die Befehlszeile eingeben:

    ```
   python system-prompt.py
    ```

1. Abschließend versuchen wir, einen Fehler auszulösen, indem wir eine Eingabeaufforderung mit **zu viele Token** ausführen:

    ```
   code error-prompt.py
    ```

1. **Führen Sie das Skript aus**, indem Sie den folgenden Befehl in die Befehlszeile eingeben: Bitte beachten Sie, dass es **sehr wahrscheinlich zu einem Fehler kommt!**

    ```
   python error-prompt.py
    ```

Nachdem Sie nun mit dem Modell interagiert haben, können Sie die Daten in Azure Monitor überprüfen.

> **Hinweis**: Es kann einige Minuten dauern, bis Überwachungsdaten in Azure Monitor angezeigt werden.

## Anzeigen von Überwachungsdaten in Azure Monitor

Um Daten anzuzeigen, die aus Ihren Modellinteraktionen gesammelt werden, greifen Sie auf das Dashboard zu, das mit einer Arbeitsmappe in Azure Monitor verknüpft ist.

### Navigieren Sie im Azure KI-Foundry-Portal zu Azure Monitor.

1. Navigieren Sie in Ihrem Browser zu der Registerkarte, auf der das **Azure AI Foundry-Portal** geöffnet ist.
1. Wählen Sie auf der linken Seite das Menü **Ablaufverfolgung** aus.
1. Wählen Sie oben den Link „**Sehen Sie sich Ihr Dashboard mit Einblicken für generative KI-Anwendungen an**“ aus. Der Link öffnet Azure Monitor auf einer neuen Registerkarte.
1. Überprüfen Sie die **Übersicht** mit zusammengefassten Daten zu den Interaktionen mit Ihrem bereitgestellten Modell.

## Interpretieren von Überwachungsmetriken in Azure Monitor

Nun ist es der Moment gekommen, sich mit den Daten auseinanderzusetzen und mit der Interpretation zu beginnen.

### Überprüfen der Tokenverwendung

Konzentrieren Sie sich zuerst auf den Abschnitt **Tokenverwendung** und überprüfen Sie die folgenden Metriken:

- **Prompt-Tokens**: Die Gesamtzahl der Tokens, die in der Eingabe (die von Ihnen gesendeten Prompts) über alle Modellaufrufe hinweg verwendet wurden.

> Betrachten Sie dies als die *Kosten für das Stellen einer Frage* an das Modell.

- **Abschlusstoken**: Die Anzahl der Token, die das Modell als Ausgabe zurückgegeben hat, im Wesentlichen die Länge der Antworten.

> Die generierten Abschlusstoken stellen häufig den Großteil der Tokenverwendung und -kosten dar, insbesondere für lange oder ausführliche Antworten.

- **Gesamttoken**: Die kombinierten Gesamtaufforderungstoken und Abschlusstoken.

> Wichtigste Metrik für Abrechnung und Leistung, da sie Latenz und Kosten steuert.

- **Gesamtaufrufe**: Die Anzahl der separaten Ableitungsanforderungen. Dies ist, wie oft das Modell aufgerufen wurde.

> Nützlich für die Analyse des Durchsatzes und verständnis der durchschnittlichen Kosten pro Anruf.

### Vergleichen der einzelnen Eingabeaufforderungen

Scrollen Sie nach unten, um die **Gen KI Spans** zu finden, die als Tabelle dargestellt wird, in der jede Eingabeaufforderung als neue Datenzeile dargestellt wird. Überprüfen und vergleichen Sie den Inhalt der folgenden Spalten:

- **Status**: Gibt an, ob ein Modellaufruf erfolgreich war oder fehlgeschlagen ist.

> Verwenden Sie diese Option, um problematische Eingabeaufforderungen oder Konfigurationsfehler zu identifizieren. Die letzte Eingabeaufforderung ist wahrscheinlich fehlgeschlagen, weil die Eingabeaufforderung zu lang war.

- **Dauer**: Anzeige in Millisekunden, wie lange es dauerte, bis das Modell reagiert hat.

> Vergleichen Sie zeilenübergreifend, um zu untersuchen, welche Aufforderungsmuster zu längeren Verarbeitungszeiten führen.

- **Eingabe**: Zeigt die Benutzernachricht an, die an das Modell gesendet wurde.

> Verwenden Sie diese Spalte, um zu bewerten, welche Promptformulierungen effizient oder problematisch sind.

- **System**: Zeigt die in der Eingabeaufforderung verwendete Systemmeldung an (sofern vorhanden).

> Vergleichen Sie Einträge, um die Auswirkungen der Verwendung oder Änderung von Systemmeldungen zu bewerten.

- **Ausgabe**: Enthält die Antwort des Modells.

> Verwenden Sie sie, um Ausführlichkeit, Relevanz und Konsistenz zu bewerten. Insbesondere im Verhältnis zur Tokenanzahl und Dauer.

## (OPTIONAL) Erstellen einer Warnung

Wenn Sie über zusätzliche Zeit verfügen, richten Sie eine Benachrichtigung ein, die Sie informiert, wenn die Modelllatenz einen bestimmten Schwellenwert überschreitet. Dies ist eine Übung, die Sie herausfordern soll, daher sind die Anweisungen bewusst weniger detailliert gehalten.

- Erstellen Sie in Azure Monitor eine **neue Warnungsregel** für Ihr Azure KI Foundry-Projekt und -Modell.
- Wählen Sie eine Metrik wie **Anforderungsdauer (ms)** aus, und definieren Sie einen Schwellenwert (z. B. größer als 4000 ms).
- Erstellen Sie eine **neue Aktionsgruppe** , um zu definieren, wie Sie benachrichtigt werden wollen.

Warnmeldungen unterstützen Sie bei der Vorbereitung der Produktion, indem sie eine proaktive Überwachung ermöglichen. Die von Ihnen konfigurierten Warnmeldungen hängen von den Prioritäten Ihres Projekts und davon ab, wie Ihr Team Risiken messen und mindern möchte.

## Wo finde ich andere Übungsszenarien?

Weitere Übungen und Übungsaufgaben finden Sie im [Azure KI-Foundry-Lernportal](https://ai.azure.com) oder im **Übungsteil** des Kurses.
