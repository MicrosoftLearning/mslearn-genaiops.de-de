---
lab:
  title: Vergleichen von Sprachmodellen aus dem Modellkatalog
  description: 'Hier erfahren Sie, wie Sie geeignete Modelle für Ihr Projekt für generative KI vergleichen und auswählen.'
---

## Vergleichen von Sprachmodellen aus dem Modellkatalog

Wenn Sie Ihren Anwendungsfall definiert haben, können Sie mithilfe des Modellkatalogs untersuchen, ob ein KI-Modell Ihr Problem löst. Sie können den Modellkatalog verwenden, um Modelle für die Bereitstellung auszuwählen, die Sie dann vergleichen können, um zu erkunden, welches Modell Ihren Anforderungen am besten entspricht.

In dieser Übung vergleichen Sie zwei Sprachmodelle über den Modellkatalog im Azure AI Foundry-Portal.

Diese Übung dauert ungefähr **30** Minuten.

## Szenario

Stellen Sie sich vor, Sie möchten eine App erstellen, mit der Teilnehmende erfahren, wie man in Python programmiert. In der App möchten Sie einen automatischen Tutor, der den Teilnehmenden beim Schreiben und Auswerten von Code helfen kann. In einer Übung müssen die Teilnehmenden den erforderlichen Python-Code erstellen, um ein Tortendiagramm zu erstellen, das auf dem folgenden Beispielbild basiert:

![Kreisdiagramm mit den in einer Prüfung erzielten Noten, aufgeschlüsselt nach den Fächern Mathematik (34,9 %), Physik (28,6 %), Chemie (20,6 %) und Englisch (15,9 %)](./images/demo.png)

Sie müssen ein Sprachmodell auswählen, das Bilder als Eingabe akzeptiert und in der Lage ist, präzisen Code zu generieren. Die verfügbaren Modelle, die diese Kriterien erfüllen, sind GPT-4 Turbo, GPT-4o und GPT-4o mini.

Beginnen wir mit der Bereitstellung der erforderlichen Ressourcen für die Arbeit mit diesen Modellen im Azure AI Foundry-Portal.

## Erstellen eines Azure KI-Hubs und eines Projekts

Sie können einen Azure KI-Hub erstellen und manuell über das Azure AI Foundry-Portal projizieren sowie die in der Übung verwendeten Modelle bereitstellen. Sie können diesen Prozess jedoch auch mithilfe einer Vorlagenanwendung mit [Azure Developer CLI (azd)](https://aka.ms/azd) automatisieren.

1. Öffnen Sie in einem Webbrowser das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` und melden Sie sich mit Ihren Azure-Anmeldeinformationen an.

1. Verwenden Sie die Taste **[\>_]** rechts neben der Suchleiste oben auf der Seite, um eine neue Cloud Shell im Azure-Portal zu erstellen, und wählen Sie eine ***PowerShell***-Umgebung aus. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

    > **Hinweis**: Wenn Sie zuvor eine Cloud-Shell erstellt haben, die eine *Bash*-Umgebung verwendet, wechseln Sie zu ***PowerShell***.

1. Wählen Sie in der Cloud Shell-Symbolleiste im Menü **Einstellungen** die Option **Zur klassischen Version wechseln**.

    **<font color="red">Stellen Sie sicher, dass Sie zur klassischen Version der Cloud Shell gewechselt haben, bevor Sie fortfahren.</font>**

1. Geben Sie im PowerShell-Fenster die folgenden Befehle ein, um das Repository dieser Übung zu klonen:

     ```powershell
    rm -r mslearn-genaiops -f
    git clone https://github.com/MicrosoftLearning/mslearn-genaiops
     ```

1. Nachdem das Repository geklont wurde, geben Sie die folgenden Befehle ein, um die Starter-Vorlage zu initialisieren. 
   
     ```powershell
    cd ./mslearn-genaiops/Starter
    azd init
     ```

1. Geben Sie der neuen Umgebung nach Aufforderung einen Namen, da dieser als Grundlage für die Vergabe eindeutiger Namen für alle bereitgestellten Ressourcen verwendet wird.
        
1. Geben Sie als Nächstes den folgenden Befehl ein, um die Starter-Vorlage auszuführen. Sie wird einen KI-Hub mit abhängigen Ressourcen, ein KI-Projekt, KI-Dienste und einen Online-Endpunkt bereitstellen. Außerdem werden die Modelle GPT-4 Turbo, GPT-4o und GPT-4o mini bereitgestellt.

     ```powershell
    azd up
     ```

1. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten, und wählen Sie dann einen der folgenden Speicherorte für die Ressourcenbereitstellung aus:
   - East US
   - USA (Ost) 2
   - USA Nord Mitte
   - USA Süd Mitte
   - Schweden, Mitte
   - USA (Westen)
   - USA, Westen 3
    
1. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 10 Minuten, kann aber in manchen Fällen auch länger dauern.

    > **Hinweis**: Azure OpenAI-Ressourcen werden auf Mandantenebene durch regionale Kontingente eingeschränkt. Die oben aufgeführten Regionen enthalten das Standardkontingent für die in dieser Übung verwendeten Modelltypen. Durch die zufällige Auswahl einer Region wird das Risiko reduziert, dass eine einzelne Region ihre Kontingentgrenze erreicht. Falls eine Kontingentgrenze erreicht wird, müssen Sie möglicherweise eine weitere Ressourcengruppe in einer anderen Region erstellen. Erfahren Sie mehr über die [Modellverfügbarkeit pro Region](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models?tabs=standard%2Cstandard-chat-completions#global-standard-model-availability)

    <details>
      <summary><b>Tipp zur Fehlerbehebung</b>: In einer bestimmten Region ist kein Kontingent verfügbar</summary>
        <p>Wenn Sie für eines der Modelle eine Fehlermeldung erhalten, weil in der von Ihnen gewählten Region kein Kontingent verfügbar ist, versuchen Sie, die folgenden Befehle auszuführen:</p>
        <ul>
          <pre><code>azd env set AZURE_ENV_NAME new_env_name
   azd env set AZURE_RESOURCE_GROUP new_rg_name
   azd env set AZURE_LOCATION new_location
   azd up</code></pre>
        Ersetzen Sie <code>new_env_name</code>, <code>new_rg_name</code> und <code>new_location</code> durch neue Werte. Der neue Standort muss sich in einer der Regionen befinden, die zu Beginn der Übung angegeben wurden, z. B. <code>eastus2</code>, <code>northcentralus</code> usw.
        </ul>
    </details>

## Vergleichen von Modellen

Sie wissen, dass es drei Modelle gibt, die Bilder als Eingabe akzeptieren, deren Rückschlussinfrastruktur vollständig von Azure verwaltet wird. Jetzt müssen Sie sie vergleichen, um zu entscheiden, welches für unseren Anwendungsfall ideal ist.

1. Öffnen Sie auf einer neuen Browserregisterkarte das [Azure AI Foundry-Portal](https://ai.azure.com) unter `https://ai.azure.com`, und melden Sie sich mit Ihren Azure-Anmeldeinformationen an.
1. Wenn Sie dazu aufgefordert werden, wählen Sie das zuvor erstellte KI-Projekt aus.
1. Navigieren Sie über das Menü auf der linken Seite zur Seite **Modellkatalog**.
1. Wählen Sie **Modelle vergleichen** aus (die Schaltfläche finden Sie neben den Filtern im Suchbereich).
1. Entfernen Sie die ausgewählten Modelle.
1. Fügen Sie einmal die drei Modelle hinzu, die Sie vergleichen möchten: **GPT-4**, **GPT-4o** und **GPT-4o-mini**. Für **GPT-4** stellen Sie sicher, dass die ausgewählte Version **turbo-2024-04-09** ist, da dies die einzige Version ist, die Bilder als Eingabe akzeptiert.
1. Ändern Sie die X-Achse in **Genauigkeit**.
1. Stellen Sie sicher, dass die Y-Achse auf **Kosten** festgelegt ist.

Überprüfen Sie die Zeichnung, und versuchen Sie, die folgenden Fragen zu beantworten:

- *Welches Modell ist genauer?*
- *Welches Modell ist günstiger zu verwenden?*

Die Genauigkeit der Benchmark-Metrik wird auf der Grundlage öffentlich verfügbarer generischer Datensätze berechnet. Aus der Grafik können wir bereits eines der Modelle herausfiltern, da es die höchsten Kosten pro Token, aber nicht die höchste Genauigkeit aufweist. Bevor Sie eine Entscheidung treffen, sollten wir die Qualität der Ergebnisse der beiden verbleibenden Modelle untersuchen, die für Ihren Anwendungsfall spezifisch sind.

## Einrichten Ihrer Entwicklungsumgebung in Cloud Shell

Zum schnellen Experimentieren und Durchlaufen verwenden Sie eine Reihe von Python-Skripts in Cloud Shell.

1. Navigieren Sie auf der Registerkarte „Azure-Portal“ zu der Ressourcengruppe, die zuvor vom Bereitstellungsskript erstellt wurde, und wählen Sie Ihre **Azure AI Foundry**-Ressource aus.
1. Wählen Sie auf der Seite **Übersicht** für Ihre Ressource **Klicken Sie hier, um Endpunkte anzuzeigen** aus und kopieren Sie den AI Foundry-API-Endpunkt.
1. Speichern Sie den Endpunkt in einem Editor. Er wird verwendet, um eine Verbindung mit Ihrem Projekt in einer Clientanwendung herzustellen.
1. Navigieren Sie zurück zur Registerkarte des Azure Portals, öffnen Sie Cloud Shell, wenn Sie den Dienst zuvor geschlossen haben, und führen Sie den folgenden Befehl aus, um zu dem Ordner mit den in dieser Übung verwendeten Codedateien zu navigieren:

     ```powershell
    cd ~/mslearn-genaiops/Files/02/
     ```

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects openai matplotlib
    ```

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu öffnen:

    ```powershell
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei den Platzhalter **your_project_endpoint** durch den zuvor kopierten Endpunkt für Ihr Projekt. Beachten Sie, dass in der Übung verwendet werden das erste und das zweite Modell verwendet werden: **gpt-4o** bzw. **gpt-4o-mini**.
1. *Nachdem* Sie den Platzhalter ersetzt haben, verwenden Sie im Code-Editor den Befehl **STRG+S**, oder **klicken Sie mit der rechten Maustaste und klicken dann auf „Speichern“**, um Ihre Änderungen zu speichern. Verwenden Sie dann den Befehl **STRG+Q**, oder **klicken Sie mit der rechten Maustaste und klicken dann auf „Beenden“**, um den Code-Editor zu schließen, während die Cloud Shell-Befehlszeile geöffnet bleibt.

## Senden von Prompts an Ihre bereitgestellten Modelle

Sie führen nun mehrere Skripts aus, die verschiedene Prompts an Ihre bereitgestellten Modelle senden. Diese Interaktionen generieren Daten, die Sie später in Azure Monitor beobachten können.

1. Führen Sie den folgenden Befehl aus, um **das erste Skript** anzuzeigen, das bereitgestellt wurde:

    ```powershell
   code model1.py
    ```

Das Skript codiert das in dieser Übung verwendete Bild in einer Daten-URL. Diese URL wird verwendet, um das Bild direkt in die Chatvervollständigungsanforderung zusammen mit dem ersten Textprompt einzubetten. Als Nächstes gibt das Skript die Antwort des Modells aus, fügt es dem Chatverlauf hinzu und sendet dann einen zweiten Prompt. Der zweite Prompt wird übermittelt und gespeichert, um die später überwachten Metriken aussagekräftiger zu machen. Sie können jedoch die Auskommentierung des optionalen Abschnitts des Codes aufheben, um die zweite Antwort ebenfalls als Ausgabe zu erhalten.

1. Geben Sie im Befehlszeilenbereich der Cloud-Shell den folgenden Befehl ein, um sich bei Azure anzumelden.

    ```
   az login
    ```

    **<font color="red">Sie müssen sich bei Azure anmelden - auch wenn die Cloud-Shell-Sitzung bereits authentifiziert ist.</font>**

    > **Hinweis**: In den meisten Szenarien ist nur die Verwendung von *az login* ausreichend. Wenn Sie jedoch Abonnements in mehreren Mandqanten haben, müssen Sie möglicherweise den Mandanten mit dem Parameter *--tenant* angeben. Weitere Informationen finden Sie unter [Interaktive Anmeldung bei Azure mit der Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Wenn Sie dazu aufgefordert werden, folgen Sie den Anweisungen, um die Anmeldeseite in einer neuen Registerkarte zu öffnen, und geben Sie den angegebenen Authentifizierungscode und Ihre Azure-Anmeldeinformationen ein. Schließen Sie dann den Anmeldevorgang in der Befehlszeile ab, und wählen Sie das Abonnement aus, das Ihren Azure AI Foundry Hub enthält, wenn Sie dazu aufgefordert werden.
1. Geben Sie nach der Anmeldung den folgenden Befehl ein, um die Anwendung auszuführen:

    ```powershell
   python model1.py
    ```

    Das Modell generiert eine Antwort, die mit Application Insights für eine weitere Analyse erfasst wird. Verwenden Sie nun das zweite Modell, um die Unterschiede zu untersuchen.

1. Geben Sie im Cloud Shell-Befehlszeilenfenster unterhalb des Code-Editors den folgenden Befehl ein, um das **zweite** Skript auszuführen:

    ```powershell
   python model2.py
    ```

    Sie haben nun Ausgaben aus beiden Modellen: Unterscheiden sie sich in irgendeiner Weise?

    > **Hinweis:** Optional können Sie die als Antworten angegebenen Skripts testen, indem Sie die Codeblöcke kopieren, den Befehl `code your_filename.py` ausführen, den Code in den Editor einfügen, die Datei speichern und dann den Befehl `python your_filename.py` ausführen. Wenn das Skript erfolgreich ausgeführt wurde, sollten Sie über ein gespeichertes Bild verfügen, das mit `download imgs/gpt-4o.jpg` oder `download imgs/gpt-4o-mini.jpg` heruntergeladen werden kann.

## Vergleichen der Tokenverwendung von Modellen

Schließlich führen Sie ein drittes Skript aus, das die Anzahl der verarbeiteten Token im Laufe der Zeit für jedes Modell darstellt. Diese Daten werden aus Azure Monitor abgerufen.

1. Bevor Sie das letzte Skript ausführen, müssen Sie die Ressourcen-ID für Ihre Azure AI Foundry-Ressource aus dem Azure-Portal kopieren. Wechseln Sie zur Übersichtsseite Ihrer Azure AI Foundry-Ressource, und wählen Sie **JSON-Ansicht** aus. Kopieren Sie die Ressourcen-ID, und ersetzen Sie den Platzhalter `your_resource_id` in der Codedatei:

    ```powershell
   code plot.py
    ```

1. Speichern Sie die Änderungen.

1. Geben Sie im Cloud Shell-Befehlszeilenfenster unterhalb des Code-Editors den folgenden Befehl ein, um das **dritte** Skript auszuführen:

    ```powershell
   python plot.py
    ```

1. Geben Sie nach Abschluss des Skripts den folgenden Befehl ein, um den Metrikplot herunterzuladen:

    ```powershell
   download imgs/plot.png
    ```

## Zusammenfassung

Nachdem Sie den Plot überprüft und sich die Benchmarkwerte im Diagramm zu Genauigkeit und Kosten gemerkt haben: Können Sie feststellen, welches Modell am besten für Ihren Anwendungsfall geeignet ist? Überwiegt der Unterschied in der Genauigkeit der Ausgaben den Unterschied in den generierten Token und damit den Kosten?

## Bereinigen

Wenn Sie mit der Erkundung von Azure KI Services fertig sind, sollten Sie die in dieser Übung erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zur Browserregisterkarte mit dem Azure-Portal zurück (oder öffnen Sie das [Azure-Portal](https://portal.azure.com?azure-portal=true) auf einer neuen Browserregisterkarte erneut), und zeigen Sie den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.
1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.
1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
