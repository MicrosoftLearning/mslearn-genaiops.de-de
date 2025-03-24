---
lab:
  title: Erkunden des Prompt Engineerings mit Prompty
---

## Erkunden des Prompt Engineerings mit Prompty

Während der Ideenfindung möchten Sie verschiedene Prompts mit Ihrem Sprachmodell schnell testen und verbessern. Es gibt verschiedene Möglichkeiten, wie Sie Prompt Engineering nutzen können: über den Playground im Azure AI Foundry-Portal oder mit Prompty für einen eher Code-First-orientierten Ansatz.

In dieser Übung erkunden Sie die Prompt-Programmierung mit Prompty in Visual Studio Code anhand eines Modells, das über Azure AI Foundry bereitgestellt wird.

Diese Übung dauert ungefähr **40** Minuten.

## Szenario

Stellen Sie sich vor, Sie möchten eine App erstellen, mit der Teilnehmende erfahren, wie man in Python programmiert. In der App möchten Sie einen automatischen Tutor, der den Teilnehmenden beim Schreiben und Auswerten von Code helfen kann. Sie möchten jedoch nicht, dass die Chat-App nur alle Antworten bereitstellt. Sie möchten, dass die Teilnehmenden individuelle Tipps erhalten, die sie zum Nachdenken über das weitere Vorgehen anregen.

Sie haben ein GPT-4-Modell ausgewählt, mit dem Sie experimentieren möchten. Sie möchten nun Prompt Engineering anwenden, um das Verhalten des Chats so zu steuern, dass er als Tutor fungiert, der personalisierte Hinweise generiert.

Beginnen wir mit der Bereitstellung der erforderlichen Ressourcen für die Arbeit mit diesem Modell im Azure AI Foundry-Portal.

## Erstellen eines Azure KI-Hubs und eines Projekts

> **Hinweis**: Wenn Sie bereits über einen Azure KI-Hub und ein Projekt verfügen, können Sie dieses Verfahren überspringen und Ihr vorhandenes Projekt verwenden.

Sie können einen Azure AI-Hub und ein Projekt manuell über das Azure AI Foundry-Portal erstellen und das in der Übung verwendete Modell bereitstellen. Sie können diesen Prozess jedoch auch mithilfe einer Vorlagenanwendung mit [Azure Developer CLI (azd)](https://aka.ms/azd) automatisieren.

1. Öffnen Sie in einem Webbrowser das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` und melden Sie sich mit Ihren Azure-Anmeldeinformationen an.

1. Verwenden Sie die Taste **[\>_]** rechts neben der Suchleiste oben auf der Seite, um eine neue Cloud Shell im Azure-Portal zu erstellen, und wählen Sie eine ***PowerShell***-Umgebung aus. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

    > **Hinweis**: Wenn Sie zuvor eine Cloud-Shell erstellt haben, die eine *Bash*-Umgebung verwendet, wechseln Sie zu ***PowerShell***.

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
        
1. Geben Sie als Nächstes den folgenden Befehl ein, um die Starter-Vorlage auszuführen. Sie wird einen KI-Hub mit abhängigen Ressourcen, ein KI-Projekt, KI-Dienste und einen Online-Endpunkt bereitstellen.

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

1. Nachdem alle Ressourcen bereitgestellt wurden, verwenden Sie die folgenden Befehle, um den Endpunkt abzurufen und auf Ihre KI Services-Ressource zuzugreifen. Beachten Sie, dass Sie `<rg-env_name>` und `<aoai-xxxxxxxxxx>` durch die Namen Ihrer Ressourcengruppe und der KI Services-Ressource ersetzen müssen. Beide werden im Ergebnis der Bereitstellung aufgeführt.

     ```powershell
    Get-AzCognitiveServicesAccount -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property endpoint
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Kopieren Sie diese Werte, da sie später verwendet werden.
   
## Einrichten Ihrer lokalen Entwicklungsumgebung

Um schnell zu experimentieren und zu iterieren, verwenden Sie Prompty in Visual Studio (VS) Code. Lassen Sie uns VS Code für die lokale Ideenfindung vorbereiten.

1. Öffnen Sie den VS Code und **klonen Sie** das folgende Git-Repository: [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git)
1. Speichern Sie den Klon auf einem lokalen Laufwerk, und öffnen Sie den Ordner nach dem Klonen.
1. Suchen und installieren Sie im Erweiterungsbereich von VS Code die Erweiterung **Prompty**.
1. Klicken Sie im VS Code Explorer (linker Bereich) mit der rechten Maustaste auf den Ordner **Files/03**.
1. Wählen Sie im Dropdownmenü **Neues Prompty** aus.
1. Öffnen Sie die neu erstellte Datei mit dem Namen **basic.prompty**.
1. Starten Sie die Prompty-Datei, indem Sie die Schaltfläche **Abspielen** in der oberen rechten Ecke wählen (oder F5 drücken).
1. Wenn Sie aufgefordert werden, sich anzumelden, wählen Sie **Zulassen**.
1. Wählen Sie Ihr Azure-Konto aus und melden Sie sich an.
1. Gehen Sie zurück zu VS Code, wo sich ein Fensterbereich **Output** mit einer Fehlermeldung öffnet. Die Fehlermeldung sollte Ihnen mitteilen, dass das bereitgestellte Modell nicht angegeben ist oder nicht gefunden werden kann.

Um den Fehler zu beheben, müssen Sie ein Modell für die Verwendung von Prompty konfigurieren.

## Aktualisieren von Promptmetadaten

Um die Prompty-Datei auszuführen, müssen Sie das Sprachmodell angeben, das für die Generierung der Antwort verwendet werden soll. Die Metadaten werden im *Frontmatter* der Prompty-Datei definiert. Aktualisieren wir die Metadaten mit der Modellkonfiguration und anderen Informationen.

1. Öffnen Sie den Visual Studio Code-Terminalbereich.
1. Kopieren Sie die Datei **basic.prompty** (in denselben Ordner) und benennen Sie die Kopie in `chat-1.prompty` um.
1. Öffnen Sie **chat-1.prompty** und aktualisieren Sie die folgenden Felder, um einige grundlegende Informationen zu ändern:

    - **Name:**

        ```yaml
        name: Python Tutor Prompt
        ```

    - **Beschreibung**:

        ```yaml
        description: A teaching assistant for students wanting to learn how to write and edit Python code.
        ```

    - **Bereitgestelltes Modell**:

        ```yaml
        azure_deployment: ${env:AZURE_OPENAI_CHAT_DEPLOYMENT}
        ```

1. Als Nächstes fügen Sie den folgenden Platzhalter für den API-Schlüssel unter dem Parameter **azure_deployment** ein.

    - **Endpunktschlüssel:**

        ```yaml
        api_key: ${env:AZURE_OPENAI_API_KEY}
        ```

1. Speichern Sie die aktualisierte Prompty-Datei.

Die Prompty-Datei verfügt jetzt über alle erforderlichen Parameter, aber einige Parameter verwenden Platzhalter, um die erforderlichen Werte abzurufen. Die Platzhalter werden in der **.env-Datei** im selben Ordner gespeichert.

## Updatemodellkonfiguration

Um anzugeben, welches Modell Prompty verwendet, müssen Sie die Informationen Ihres Modells in der env-Datei angeben.

1. Öffnen Sie die Datei **.env** im Ordner **Files/03**.
1. Aktualisieren Sie jeden Platzhalter mit den Werten, die Sie zuvor aus der Ausgabe der Modellbereitstellung im Azure-Portal kopiert haben:

    ```yaml
    - AZURE_OPENAI_CHAT_DEPLOYMENT="gpt-4"
    - AZURE_OPENAI_ENDPOINT="<Your endpoint target URI>"
    - AZURE_OPENAI_API_KEY="<Your endpoint key>"
    ```

1. Speichern Sie die ENV-Datei.
1. Führen Sie die Datei **chat-1.prompty** erneut aus.

Sie sollten nun eine KI-generierte Antwort erhalten, die jedoch nichts mit Ihrem Szenario zu tun hat, da sie nur die Beispieleingabe verwendet. Aktualisieren wir die Vorlage, um sie zu einem KI-Lehrassistenten zu machen.

## Bearbeiten des Beispielabschnitts

Der Beispielabschnitt gibt die Eingaben für das Prompty an und liefert Standardwerte, die verwendet werden, wenn keine Eingaben gemacht werden.

1. Bearbeiten Sie die Felder der folgenden Parameter:

    - **firstName**: Wählen Sie einen anderen Namen aus.
    - **context**: Entfernen Sie diesen gesamten Abschnitt.
    - **question**: Ersetzen Sie den bereitgestellten Text durch:

    ```yaml
    What is the difference between 'for' loops and 'while' loops?
    ```

    Ihr **Beispielabschnitt** sollte nun so aussehen:
    
    ```yaml
    sample:
    firstName: Daniel
    question: What is the difference between 'for' loops and 'while' loops?
    ```

    1. Führen Sie die aktualisierte Prompty-Datei aus und überprüfen Sie die Ausgabe.

