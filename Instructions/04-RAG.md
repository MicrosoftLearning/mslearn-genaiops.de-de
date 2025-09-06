---
lab:
  title: Orchestrieren eines RAG-Systems
  description: 'Hier erfahren Sie, wie Sie RAG-Systeme (Retrieval Augmented Generation) in Ihren Apps implementieren, um die Genauigkeit und Relevanz generierter Antworten zu verbessern.'
---

## Orchestrieren eines RAG-Systems

RAG-Systeme (Retrieval-Augmented Generation) kombinieren die Leistungsfähigkeit großer Sprachmodelle mit effizienten Abrufmechanismen, um die Genauigkeit und Relevanz der generierten Antworten zu verbessern. Durch die Nutzung von LangChain für die Orchestrierung und Azure AI Foundry für KI-Funktionen können wir eine robuste Pipeline erstellen, die relevante Informationen aus einem Datensatz abruft und kohärente Antworten generiert. In dieser Übung werden Sie die Schritte zum Einrichten Ihrer Umgebung, zum Vorverarbeiten von Daten, zum Erstellen von Einbettungen und zum Erstellen eines Index durchlaufen, um letztendlich ein RAG-System effektiv zu implementieren.

Diese Übung dauert ungefähr **30** Minuten.

## Szenario

Stellen Sie sich vor, Sie möchten eine App erstellen, die Empfehlungen zu Hotels in London gibt. In der App benötigen Sie einen Agent, der nicht nur Hotels empfehlen, sondern auch Fragen der Benutzenden zu den Hotels beantworten kann.

Sie haben ein GPT-4-Modell ausgewählt, um generative Antworten bereitzustellen. Sie möchten nun ein RAG-System zusammenstellen, das dem Modell auf der Grundlage von Nutzerbewertungen grundlegende Daten zur Verfügung stellt und das Verhalten des Chats so steuert, dass personalisierte Empfehlungen gegeben werden.

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
    cd ~/mslearn-genaiops/Files/04/
     ```

1. Geben Sie die folgenden Befehle ein, um eine virtuelle Umgebung zu aktivieren und die benötigten Bibliotheken zu installieren:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv langchain-text-splitters langchain-community langchain-openai
    ```

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu öffnen:

    ```powershell
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei die Platzhalter **your_azure_openai_service_endpoint** und **your_azure_openai_service_api_key** durch den Endpunkt und die Schlüsselwerte, die Sie zuvor kopiert haben.
1. *Nachdem* Sie die Platzhalter ersetzt haben, verwenden Sie im Code-Editor den Befehl **STRG+S**, oder **klicken Sie mit der rechten Maustaste und klicken dann auf „Speichern“**, um Ihre Änderungen zu speichern. Verwenden Sie dann den Befehl **STRG+Q**, oder **klicken Sie mit der rechten Maustaste und klicken dann auf „Beenden“**, um den Code-Editor zu schließen, während die Cloud Shell-Befehlszeile geöffnet bleibt.

## Implementieren von RAG

Sie führen nun ein Skript aus, das Daten erfasst und vorverarbeitet, Einbettungen erstellt und einen Vektorspeicher und -index erstellt, wodurch Sie letztendlich ein RAG-System effektiv implementieren können.

1. Führen Sie den folgenden Befehl aus, um das bereitgestellte **Skript zu bearbeiten**:

    ```powershell
   code RAG.py
    ```

1. Suchen Sie im Skript nach **# Initialize the components that will be used from LangChain's suite of integrations**. Unter diesem Kommentar fügen Sie den folgenden Code ein:

    ```python
   # Initialize the components that will be used from LangChain's suite of integrations
   llm = AzureChatOpenAI(azure_deployment=llm_name)
   embeddings = AzureOpenAIEmbeddings(azure_deployment=embeddings_name)
   vector_store = InMemoryVectorStore(embeddings)
    ```

1. Überprüfen Sie das Skript, und beachten Sie, dass es eine CSV-Datei mit Hotelbewertungen als Groundingdaten verwendet. Sie können den Inhalt dieser Datei anzeigen, indem Sie den Befehl `download app_hotel_reviews.csv` im Befehlszeilenbereich ausführen und die Datei öffnen.
1. Suchen Sie als Nächstes **# Split the documents into chunks for embedding and vector storage**. Unter diesem Kommentar fügen Sie den folgenden Code ein:

    ```python
   # Split the documents into chunks for embedding and vector storage
   text_splitter = RecursiveCharacterTextSplitter(
       chunk_size=200,
       chunk_overlap=20,
       add_start_index=True,
   )
   all_splits = text_splitter.split_documents(docs)
    
   print(f"Split documents into {len(all_splits)} sub-documents.")
    ```

    Der obige Code teilt eine Reihe großer Dokumente in kleinere Blöcke auf. Dies ist wichtig, da viele Einbettungsmodelle (z. B. die für semantische Suche oder Vektordatenbanken) ein Tokenlimit aufweisen und bei kürzeren Texten besser funktionieren.

1. Suchen Sie als Nächstes **# Embed the contents of each text chunk and insert these embeddings into a vector store**. Unter diesem Kommentar fügen Sie den folgenden Code ein:

    ```python
   # Embed the contents of each text chunk and insert these embeddings into a vector store
   document_ids = vector_store.add_documents(documents=all_splits)
    ```

1. Suchen Sie als Nächstes **# Retrieve relevant documents from the vector store based on user input**. Fügen Sie unter diesem Kommentar den folgenden Code ein. Beachten Sie dabei den richtigen Einzug:

    ```python
   # Retrieve relevant documents from the vector store based on user input
   retrieved_docs = vector_store.similarity_search(question, k=10)
   docs_content = "\n\n".join(doc.page_content for doc in retrieved_docs)
    ```

    Der obige Code durchsucht den Vektorspeicher nach den Dokumenten, die der Eingabefrage der benutzenden Person am meisten ähneln. Die Frage wird mithilfe des für die Dokumente verwendeten Einbettungsmodells in einen Vektor konvertiert. Anschließend vergleicht das System diesen Vektor mit allen gespeicherten Vektoren und ruft die ähnlichsten ab.

1. Speichern Sie die Änderungen.
1. **Führen Sie das Skript aus** , indem Sie den folgenden Befehl in die Befehlszeile eingeben:

    ```powershell
   python RAG.py
    ```

1. Sobald die Anwendung ausgeführt wird, können Sie Fragen wie `Where can I stay in London?` stellen und dann mit spezifischeren Anfragen fortfahren.

## Zusammenfassung

In dieser Übung haben Sie ein typisches RAG-System mit seinen Hauptkomponenten erstellt. Indem Sie Ihre eigenen Dokumente verwenden, um Informationen für die Antworten eines Modells bereitzustellen, geben Sie Groundingdaten an, die vom LLM beim Formulieren einer Antwort verwendet werden. Für eine Unternehmenslösung bedeutet dies, dass Sie generative KI auf Ihre Unternehmensinhalte beschränken können.

## Bereinigen

Wenn Sie mit der Erkundung von Azure KI Services fertig sind, sollten Sie die in dieser Übung erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zur Browserregisterkarte mit dem Azure-Portal zurück (oder öffnen Sie das [Azure-Portal](https://portal.azure.com?azure-portal=true) auf einer neuen Browserregisterkarte erneut), und zeigen Sie den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.
1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.
1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
