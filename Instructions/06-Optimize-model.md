---
lab:
  title: Optimieren des Modells mithilfe eines synthetischen Datasets
  description: 'Hier erfahren Sie, wie Sie synthetische Datasets erstellen und verwenden, um die Leistung und Zuverlässigkeit Ihres Modells zu verbessern.'
---

## Optimieren des Modells mithilfe eines synthetischen Datasets

Das Optimieren einer generativen KI-Anwendung umfasst die Nutzung von Datasets, um die Leistung und Zuverlässigkeit des Modells zu verbessern. Durch die Verwendung synthetischer Daten können Fachkräfte in der Entwicklung eine Vielzahl von Szenarien und Grenzfällen simulieren, die in realen Daten möglicherweise nicht vorhanden sind. Darüber hinaus ist die Bewertung der Ergebnisse des Modells entscheidend, um qualitativ hochwertige und zuverlässige KI-Anwendungen zu erhalten. Der gesamte Optimierungs- und Evaluierungsprozess kann mithilfe des Azure KI-Bewertungs-SDK effizient verwaltet werden, das robuste Tools und Frameworks zur Optimierung dieser Aufgaben bereitstellt.

Diese Übung dauert ungefähr **30** Minuten.\*

> \* Diese Schätzung schließt nicht die optionale Aufgabe am Ende der Übung ein.
## Szenario

Stellen Sie sich vor, Sie möchten eine KI-gesteuerte Smart-Guide-App entwickeln, um das Erlebnis für Besuchende in einem Museum zu verbessern. Die App zielt darauf ab, Fragen zu historischen Figuren zu beantworten. Um die Antworten aus der App auszuwerten, müssen Sie ein umfassendes synthetisches Frage-Antwort-Dataset erstellen, das verschiedene Aspekte dieser Persönlichkeiten und ihrer Arbeit abdeckt.

Sie haben ein GPT-4-Modell ausgewählt, um generative Antworten bereitzustellen. Sie möchten nun einen Simulator zusammenstellen, der kontextbezogene Interaktionen generiert und die Leistung der KI in verschiedenen Szenarien auswertet.

Beginnen wir mit der Bereitstellung der erforderlichen Ressourcen zum Erstellen dieser Anwendung.

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

1. Nachdem alle Ressourcen bereitgestellt wurden, verwenden Sie die folgenden Befehle, um den Endpunkt abzurufen und auf Ihre KI Services-Ressource zuzugreifen. Beachten Sie, dass Sie `<rg-env_name>` und `<aoai-xxxxxxxxxx>` durch die Namen Ihrer Ressourcengruppe und der KI Services-Ressource ersetzen müssen. Beide werden im Ergebnis der Bereitstellung aufgeführt.

     ```powershell
    Get-AzCognitiveServicesAccount -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property endpoint
     ```

     ```powershell
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Kopieren Sie diese Werte, da sie später verwendet werden.

## Einrichten Ihrer Entwicklungsumgebung in Cloud Shell

Zum schnellen Experimentieren und Durchlaufen verwenden Sie eine Reihe von Python-Skripts in Cloud Shell.

1. Geben Sie im Befehlszeilenbereich von Cloud Shell den folgenden Befehl ein, um zu dem Ordner mit den in dieser Übung verwendeten Codedateien zu navigieren:

     ```powershell
    cd ~/mslearn-genaiops/Files/06/
     ```

1. Geben Sie die folgenden Befehle ein, um eine virtuelle Umgebung zu aktivieren und die benötigten Bibliotheken zu installieren:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-ai-evaluation azure-ai-projects promptflow wikipedia aiohttp openai==1.77.0
    ```

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu öffnen:

    ```powershell
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei die Platzhalter **your_azure_openai_service_endpoint** und **your_azure_openai_service_api_key** durch den Endpunkt und die Schlüsselwerte, die Sie zuvor kopiert haben.
1. *Nachdem* Sie die Platzhalter ersetzt haben, verwenden Sie im Code-Editor den Befehl **STRG+S**, oder **klicken Sie mit der rechten Maustaste und klicken dann auf „Speichern“**, um Ihre Änderungen zu speichern. Verwenden Sie dann den Befehl **STRG+Q**, oder **klicken Sie mit der rechten Maustaste und klicken dann auf „Beenden“**, um den Code-Editor zu schließen, während die Cloud Shell-Befehlszeile geöffnet bleibt.

## Generieren synthetischer Daten

Sie führen nun ein Skript aus, das ein synthetisches Dataset generiert und es verwendet, um die Qualität Ihres vortrainierten Modells zu bewerten.

1. Führen Sie den folgenden Befehl aus, um das bereitgestellte **Skript zu bearbeiten**:

    ```powershell
   code generate_synth_data.py
    ```

1. Suchen Sie im Skript nach **# Define callback function**.
1. Unter diesem Kommentar fügen Sie den folgenden Code ein:

    ```
    async def callback(
        messages: List[Dict],
        stream: bool = False,
        session_state: Any = None,  # noqa: ANN401
        context: Optional[Dict[str, Any]] = None,
    ) -> dict:
        messages_list = messages["messages"]
        # Get the last message
        latest_message = messages_list[-1]
        query = latest_message["content"]
        context = text
        # Call your endpoint or AI application here
        current_dir = os.getcwd()
        prompty_path = os.path.join(current_dir, "application.prompty")
        _flow = load_flow(source=prompty_path)
        response = _flow(query=query, context=context, conversation_history=messages_list)
        # Format the response to follow the OpenAI chat protocol
        formatted_response = {
            "content": response,
            "role": "assistant",
            "context": context,
        }
        messages["messages"].append(formatted_response)
        return {
            "messages": messages["messages"],
            "stream": stream,
            "session_state": session_state,
            "context": context
        }
    ```

    Sie können einen beliebigen Anwendungsendpunkt für die Simulation verwenden, indem Sie eine Zielrückruffunktion angeben. In diesem Fall verwenden Sie eine Anwendung, bei dem es sich um ein LLM mit einer prompty-Datei (`application.prompty`) handelt. Die obige Rückruffunktion verarbeitet jede vom Simulator generierte Nachricht durch Ausführung der folgenden Aufgaben:
    * Abrufen der neuesten Benutzernachricht
    * Laden eines prompt flow aus „application.prompty“.
    * Generieren einer Antwort mithilfe des Prompt Flow
    * Formatieren der Antwort, damit diese dem OpenAI-Chatprotokoll entspricht
    * Anhängen der Antwort des Assistenten an die Nachrichtenliste

    >**Hinweis:** Weitere Informationen zur Verwendung von prompty finden Sie in der [Dokumentation zu Prompty](https://www.prompty.ai/docs).

1. Suchen Sie als Nächstes **# Run the simulator**.
1. Unter diesem Kommentar fügen Sie den folgenden Code ein:

    ```
    model_config = {
        "azure_endpoint": os.getenv("AZURE_OPENAI_ENDPOINT"),
        "api_key": os.getenv("AZURE_OPENAI_API_KEY"),
        "azure_deployment": os.getenv("AZURE_OPENAI_DEPLOYMENT"),
    }
    
    simulator = Simulator(model_config=model_config)
    
    outputs = asyncio.run(simulator(
        target=callback,
        text=text,
        num_queries=1,  # Minimal number of queries
    ))
    
    output_file = "simulation_output.jsonl"
    with open(output_file, "w") as file:
        for output in outputs:
            file.write(output.to_eval_qr_json_lines())
    ```

   Der obige Code initialisiert den Simulator und führt ihn aus, um synthetische Unterhaltungen basierend auf einem Text zu generieren, der zuvor aus Wikipedia extrahiert wurde.

1. Suchen Sie als Nächstes **# Evaluate the model**.
1. Unter diesem Kommentar fügen Sie den folgenden Code ein:

    ```
    groundedness_evaluator = GroundednessEvaluator(model_config=model_config)
    eval_output = evaluate(
        data=output_file,
        evaluators={
            "groundedness": groundedness_evaluator
        },
        output_path="groundedness_eval_output.json"
    )
    ```

    Nachdem Sie nun über ein Dataset verfügen, können Sie die Qualität und Effektivität Ihrer Anwendung für generative KI bewerten. Im obigen Code verwenden Sie Quellenübereinstimmung als Qualitätsmetrik.

1. Speichern Sie die Änderungen.
1. Geben Sie im Cloud Shell-Befehlszeilenfenster unterhalb des Code-Editors den folgenden Befehl ein, **um das Skript auszuführen**:

    ```
   python generate_synth_data.py
    ```

    Nachdem das Skript abgeschlossen ist, können Sie die Ausgabedateien herunterladen, indem Sie `download simulation_output.jsonl` und `download groundedness_eval_output.json` ausführen und den jeweiligen Inhalt überprüfen. Wenn die Quellenübereinstimmungsmetrik nicht nahe bei 3,0 liegt, können Sie die LLM-Parameter wie `temperature`, `top_p`, `presence_penalty` oder `frequency_penalty` in der Datei `application.prompty` ändern und das Skript erneut ausführen, um ein neues Dataset für die Auswertung zu generieren. Sie können auch `wiki_search_term` ändern, um ein synthetisches Dataset basierend auf einem anderen Kontext abzurufen.

## (OPTIONAL) Optimieren Ihres Modells

Wenn Sie noch Zeit haben, können Sie das generierte Dataset verwenden, um Ihr Modell in Azure AI Foundry zu optimieren. Die Feinabstimmung hängt von den Ressourcen der Cloud-Infrastruktur ab, deren Bereitstellung je nach Rechenzentrumskapazität und gleichzeitiger Nachfrage unterschiedlich viel Zeit in Anspruch nehmen kann.

1. Öffnen Sie eine neue Browserregisterkarte, und navigieren Sie zum [Azure AI Foundry-Portal](https://ai.azure.com) unter `https://ai.azure.com`. Melden Sie sich mit Ihren Azure-Anmeldeinformationen an.
1. Wählen Sie auf der Startseite von AI Foundry das Projekt aus, das Sie zu Beginn der Übung erstellt haben.
1. Navigieren Sie zur Seite **Feinabstimmung** unter dem Abschnitt **Erstellen und Anpassen**, indem Sie das Menü auf der linken Seite verwenden.
1. Klicken Sie auf die Schaltfläche aus, um ein neues Feinabstimmungsmodell hinzuzufügen, wählen Sie das Modell **gpt-4o** aus und klicken Sie dann auf **Weiter**.
1. **Optimieren Sie** das Modell mithilfe der folgenden Konfiguration:
    - **Modellversion**: *Wählen Sie die Standardversion aus.*
    - **Methode der Anpassung**: Überwacht
    - **Modellsuffix**: `ft-travel`
    - **Verbundene KI-Ressource**: *Wählen Sie die Verbindung, die bei der Erstellung Ihres Hubs erstellt wurde. Sollte standardmäßig ausgewählt sein.*
    - **Trainingsdaten**: Dateien hochladen

    <details>  
    <summary><b>Tip zur Problembehandlung</b>: Berechtigungsfehler</summary>
    <p>Wenn Sie einen Berechtigungsfehler erhalten, versuchen Sie Folgendes, um das Problem zu beheben:</p>
    <ul>
        <li>Wählen Sie im Ressourcenmenü des Azure-Portals AI Dienste aus.</li>
        <li>Bestätigen Sie unter Ressourcenverwaltung auf der Registerkarte Identität, dass es sich um eine vom System zugewiesene verwaltete Identität handelt.</li>
        <li>Navigieren Sie zum dazugehörigen Speicherkonto. Fügen Sie auf der IAM-Seite die Rollenzuweisung <em>Storage Blob Data Besitzender</em> hinzu.</li>
        <li>Wählen Sie unter <strong>Zugriff zuweisen an</strong> die Option <strong>Verwaltete Identität</strong>, <strong>+ Mitglieder auswählen</strong>, <strong>Alle systemseitig zugewiesenen verwalteten Identitäten</strong> und Ihre Azure KI-Ressource aus.</li>
        <li>Überprüfen und zuweisen, um die neuen Einstellungen zu speichern, und wiederholen Sie den vorherigen Schritt.</li>
    </ul>
    </details>

    - **Datei hochladen**: Wählen Sie die JSONL-Datei aus, die Sie in einem früheren Schritt heruntergeladen haben.
    - **Gültigkeitsprüfungsdaten**: Keine
    - **Vorgangsparameter**: *Standardeinstellungen beibehalten*
1. Die Feinabstimmung beginnt und kann einige Zeit in Anspruch nehmen.

    > **Hinweis**: Die Feinabstimmung und Bereitstellung kann sehr viel Zeit in Anspruch nehmen (30 Minuten oder länger). Überprüfen Sie daher in regelmäßigen Abständen Ihre Einstellungen. Sie können weitere Details über den bisherigen Fortschritt sehen, indem Sie den Feinabstimmungsmodellauftrag auswählen und die Registerkarte **Protokolle** anzeigen.

## (OPTIONAL) Bereitstellen des optimierten Modells

Wenn die Feinabstimmung erfolgreich abgeschlossen wurde, können Sie das fein abgestimmte Modell bereitstellen.

1. Klicken Sie auf den Feinabstimmungsauftragslink, um die Detailseite zu öffnen. Wählen Sie die Registerkarte **Metriken** aus und erkunden Sie die Feinabstimmung der Metriken.
1. Stellen Sie das fein abgestimmte Modell mit den folgenden Konfigurationen bereit:
    - **Bereitstellungsname:***Ein gültiger Name für Ihre Modellimplementierung*
    - **Bereitstellungstyp**: Standard
    - **Ratenbegrenzung für Token pro Minute (Tausender)**: 5.000 *(oder der maximale Wert, der in Ihrem Abonnement verfügbar ist, wenn er weniger als 5.000 beträgt)
    - **Inhaltsfilter**: Standard
1. Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie sie testen können; dies kann eine Weile dauern. Überprüfen Sie den **Bereitstellungsstatus**, bis er erfolgreich war (möglicherweise müssen Sie den Browser aktualisieren, um den aktualisierten Status zu sehen).
1. Wenn die Verteilung fertig ist, navigieren Sie zu dem fein abgestimmten Modell und wählen Sie **Open in playground**.

    Nachdem Sie nun Ihr optimiertes Modell bereitgestellt haben, können Sie es wie jedes Basismodell im Chat-Playground testen.

## Zusammenfassung

In dieser Übung haben Sie ein synthetisches Dataset erstellt, das eine Unterhaltung zwischen einer benutzenden Person und einer Chatvervollständigungs-App simuliert. Mithilfe dieses Datasets können Sie die Qualität der App-Antworten bewerten und sie optimieren, um die gewünschten Ergebnisse zu erzielen.

## Bereinigen

Wenn Sie mit der Erkundung von Azure KI Services fertig sind, sollten Sie die in dieser Übung erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zur Browserregisterkarte mit dem Azure-Portal zurück (oder öffnen Sie das [Azure-Portal](https://portal.azure.com?azure-portal=true) auf einer neuen Browserregisterkarte erneut), und zeigen Sie den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.
1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.
1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
