---
lab:
  title: Überwachen der Anwendung für generative KI
---

# Überwachen der Anwendung für generative KI

Diese Übung dauert ca. **30** Minuten.

> **Hinweis**: Diese Übung setzt gewisse Kenntnisse von Azure KI Foundry voraus. Aus diesem Grund sind einige Anweisungen bewusst weniger detailliert gehalten, um eine aktivere Erkundung und praktisches Lernen zu fördern.

## Einführung

In dieser Übung aktivieren Sie die Überwachung für eine App zum Abschließen von Chats und zeigen deren Leistung in Azure Monitor an. Sie interagieren mit Ihrem bereitgestellten Modell, um Daten zu generieren, die generierten Daten über das Dashboard „Einblicke für generative KI-Anwendungen“ anzuzeigen und Warnmeldungen einzurichten, um die Bereitstellung des Modells zu optimieren.

## 1. Einrichten der Umgebung

Um diese Übung abzuschließen benötigen Sie Folgendes:

- Ein Azure KI Foundry-Hub.
- Ein Azure KI Foundry-Projekt
- Ein bereitgestelltes Modell (z. B. GPT-4o),
- Eine verbundene Application Insights-Ressource.

### A. Erstellen eines Azure KI Foundry-Hubs und -Projekts

Um einen Hub und ein Projekt schnell einzurichten, finden Sie unten einfache Anweisungen zur Verwendung der Benutzeroberfläche des Azure KI Foundry-Portals.

1. Navigieren Sie zum Azure KI Foundry-Portal: Öffnen Sie [https://ai.azure.com](https://ai.azure.com).
1. Melden Sie sich mit Ihren Azure-Anmeldeinformationen an.
1. Erstellen eines Projekts:
    1. Navigieren Sie zu **allen Hubs + Projekten**.
    1. Wählen Sie **+ New project** aus.
    1. Geben Sie einen **Projektnamen** ein.
    1. Wenn Sie dazu aufgefordert werden, **erstellen Sie einen neuen Hub**.
    1. Anpassen des Hubs:

        1. Wählen Sie **Abonnement**, **Ressourcengruppe**, **Speicherort** usw. aus.
        1. Verbinden Sie eine **neue Azure KI Services-Ressource** (KI-Suche überspringen).

    1. Überprüfen Sie die Angaben, und wählen Sie **Erstellen** aus.

1. **Warten Sie, bis die Bereitstellung abgeschlossen ist** (~ 1–2 Minuten).

### B. Bereitstellen eines Modells

Um Daten zu generieren, die Sie überwachen können, müssen Sie zuerst ein Modell bereitstellen und damit interagieren. In den Anweisungen werden Sie aufgefordert, ein GPT-4o-Modell bereitzustellen, jedoch **können Sie jedes Modell** aus der Azure OpenAI Service-Sammlung verwenden, das Ihnen zur Verfügung steht.

1. Verwenden Sie das Menü auf der linken Seite, wählen Sie unter **Meine Assets** die Seite **Modelle + Endpunkte** aus.
1. Stellen Sie ein **Basismodell** bereit, und wählen Sie **gpt-4o** aus.
1. **Passen Sie die Bereitstellungsdetails an**.
1. Legen Sie die **Kapazität** auf **5K-Token pro Minute (TPM)** fest.

Der Hub und das Projekt sind bereit, wobei alle erforderlichen Azure-Ressourcen automatisch bereitgestellt werden.

### C. Verbinden von Application Insights

Verbinden Sie Application Insights mit Ihrem Projekt in Azure KI Foundry, um gesammelte Daten für die Überwachung zu starten.

1. Öffnen Sie Ihr Projekt im Azure KI Foundry-Portal.
1. Verwenden Sie das Menü auf der linken Seite, und wählen Sie die Seite **Ablaufverfolgung** aus.
1. **Erstellen Sie eine neue** Application Insights-Ressource, um eine Verbindung mit Ihrer App herzustellen.
1. Geben Sie den **Namen der Application Insights-Ressource** ein.

Application Insights ist jetzt mit Ihrem Projekt verbunden, und die Daten werden für die Analyse erfasst.

## 2. Interagieren mit einem bereitgestellten Modell

Sie interagieren programmgesteuert mit Ihrem bereitgestellten Modell, indem Sie eine Verbindung mit Ihrem Azure KI Foundry-Projekt mithilfe von Azure Cloud Shell einrichten. Auf diese Weise können Sie eine Eingabeaufforderung an das Modell senden und Überwachungsdaten generieren.

### A. Herstellen einer Verbindung mit einem Modell über die Cloud Shell

Beginnen Sie, indem Sie die erforderlichen Informationen abrufen, die für die Interaktion mit Ihrem Modell authentifiziert werden sollen. Anschließend greifen Sie auf die Azure Cloud Shell zu und aktualisieren die Konfiguration, um die bereitgestellten Eingabeaufforderungen an Ihr eigenes bereitgestelltes Modell zu senden.

1. Wechseln Sie im Azure AI Foundry-Portal zur **Übersichtsseite** Ihres Projekts.
1. Beachten Sie im Bereich **Projektdetails** die **Projektverbindungszeichenfolge**.
1. **Speichern Sie** die Zeichenfolge in einem Editor. Sie verwenden diese Verbindungszeichenfolge, um eine Verbindung mit Ihrem Projekt in einer Clientanwendung herzustellen.
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

1. Navigieren Sie nach dem Klonen des Repositorys zu dem Ordner, der die Codedateien der Anwendung enthält:  

    ```
   cd mslearn-genaiops/Files/07
    ```

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference azure-monitor-opentelemetry
    ```

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu öffnen:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. In der Code-Datei:

    1. Ersetzen Sie den Platzhalter **your_project_connection_string** durch die Verbindungszeichenfolge für Ihr Projekt (kopiert von der Seite **Übersicht** des Projekts im Azure KI-Foundry-Portal).
    1. Ersetzen Sie den Platzhalter **your_model_deployment** durch den Namen, den Sie Ihrer GPT-4o-Modellbereitstellung zugewiesen haben (standardmäßig `gpt-4o`).

1. *Nachdem* Sie die Platzhalter ersetzt haben, verwenden Sie im Code-Editor den Befehl **STRG+S** oder **Rechtsklick > Speichern**, um **Ihre Änderungen zu speichern**.

### B. Senden von Eingabeaufforderungen an Ihr bereitgestelltes Modell

Sie führen nun mehrere Skripts aus, die verschiedene Eingabeaufforderungen an Ihr bereitgestelltes Modell senden. Diese Interaktionen generieren Daten, die Sie später in Azure Monitor beobachten können.

1. Führen Sie den folgenden Befehl aus, um **das erste Skript** anzuzeigen, das bereitgestellt wurde:

    ```
   code start-prompt.py
    ```

1. Geben Sie im Cloud Shell-Befehlszeilenfenster unterhalb des Code-Editors den folgenden Befehl ein, **um das Skript auszuführen**:

    ```
   python start-prompt.py
    ```

    Das Modell generiert eine Antwort, die mit Application Insights für eine weitere Analyse erfasst wird. Lassen Sie uns unsere Eingabeaufforderungen variieren, um ihre Effekte zu untersuchen.

1. **Öffnen und überprüfen Sie das Skript**, wo die Eingabeaufforderung anweist, **nur mit einem Satz und einer Liste zu antworten**:

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

## 4. Überwachungsdaten in Azure Monitor anzeigen

Um Daten anzuzeigen, die aus Ihren Modellinteraktionen gesammelt werden, greifen Sie auf das Dashboard zu, das mit einer Arbeitsmappe in Azure Monitor verknüpft ist.

### A. Navigieren Sie im Azure KI-Foundry-Portal zu Azure Monitor.

1. Navigieren Sie in Ihrem Browser zu der Registerkarte, auf der das **Azure AI Foundry-Portal** geöffnet ist.
1. Wählen Sie auf der linken Seite das Menü **Ablaufverfolgung** aus.
1. Wählen Sie oben den Link „**Sehen Sie sich Ihr Dashboard mit Einblicken für generative KI-Anwendungen an**“ aus. Der Link öffnet Azure Monitor auf einer neuen Registerkarte.
1. Überprüfen Sie die **Übersicht** mit zusammengefassten Daten zu den Interaktionen mit Ihrem bereitgestellten Modell.

## 5. Interpretieren von Überwachungsmetriken in Azure Monitor

Nun ist es der Moment gekommen, sich mit den Daten auseinanderzusetzen und mit der Interpretation zu beginnen.

### A. Überprüfen der Tokenverwendung

Konzentrieren Sie sich zuerst auf den Abschnitt **Tokenverwendung** und überprüfen Sie die folgenden Metriken:

- **Prompt-Tokens**: Die Gesamtzahl der Tokens, die in der Eingabe (die von Ihnen gesendeten Prompts) über alle Modellaufrufe hinweg verwendet wurden.

> Betrachten Sie dies als die *Kosten für das Stellen einer Frage* an das Modell.

- **Abschlusstoken**: Die Anzahl der Token, die das Modell als Ausgabe zurückgegeben hat, im Wesentlichen die Länge der Antworten.

> Die generierten Abschlusstoken stellen häufig den Großteil der Tokenverwendung und -kosten dar, insbesondere für lange oder ausführliche Antworten.

- **Gesamttoken**: Die kombinierten Gesamtaufforderungstoken und Abschlusstoken.

> Wichtigste Metrik für Abrechnung und Leistung, da sie Latenz und Kosten steuert.

- **Gesamtaufrufe**: Die Anzahl der separaten Ableitungsanforderungen. Dies ist, wie oft das Modell aufgerufen wurde.

> Nützlich für die Analyse des Durchsatzes und verständnis der durchschnittlichen Kosten pro Anruf.

### B. Vergleichen der einzelnen Eingabeaufforderungen

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

## 6. (OPTIONAL) Erstellen einer Warnung

Wenn Sie über zusätzliche Zeit verfügen, richten Sie eine Benachrichtigung ein, die Sie informiert, wenn die Modelllatenz einen bestimmten Schwellenwert überschreitet. Dies ist eine Übung, die Sie herausfordern soll, daher sind die Anweisungen bewusst weniger detailliert gehalten.

- Erstellen Sie in Azure Monitor eine **neue Warnungsregel** für Ihr Azure KI Foundry-Projekt und -Modell.
- Wählen Sie eine Metrik wie **Anforderungsdauer (ms)** aus, und definieren Sie einen Schwellenwert (z. B. größer als 4000 ms).
- Erstellen Sie eine **neue Aktionsgruppe** , um zu definieren, wie Sie benachrichtigt werden wollen.

Warnmeldungen unterstützen Sie bei der Vorbereitung der Produktion, indem sie eine proaktive Überwachung ermöglichen. Die von Ihnen konfigurierten Warnmeldungen hängen von den Prioritäten Ihres Projekts und davon ab, wie Ihr Team Risiken messen und mindern möchte.

## Wo finde ich andere Übungsszenarien?

Weitere Übungen und Übungsaufgaben finden Sie im [Azure KI-Foundry-Lernportal](https://ai.azure.com) oder im **Übungsteil** des Kurses.
