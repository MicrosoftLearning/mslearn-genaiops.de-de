---
lab:
  title: Erkunden des Prompt Engineerings mit Prompty
  description: 'Hier erfahren Sie, wie Sie mithilfe von prompty verschiedene Prompts mit Ihrem Sprachmodell schnell testen und verbessern sowie sicherstellen können, dass sie so konstruiert und orchestriert sind, dass sie optimale Ergebnisse erzielen.'
---

## Erkunden des Prompt Engineerings mit Prompty

Diese Übung dauert ca. **45** Minuten.

> **Hinweis**: Diese Übung setzt gewisse Kenntnisse von Azure KI Foundry voraus. Aus diesem Grund sind einige Anweisungen bewusst weniger detailliert gehalten, um eine aktivere Erkundung und praktisches Lernen zu fördern.

## Einführung

Während der Ideenfindung möchten Sie verschiedene Prompts mit Ihrem Sprachmodell schnell testen und verbessern. Es gibt verschiedene Möglichkeiten, wie Sie Prompt Engineering nutzen können: über den Playground im Azure AI Foundry-Portal oder mit Prompty für einen eher Code-First-orientierten Ansatz.

In dieser Übung erkunden Sie das Prompt Engineering mit prompty in Visual Studio Code anhand eines Modells, das über Azure AI Foundry bereitgestellt wird.

## Einrichten der Umgebung

Um diese Übung abzuschließen benötigen Sie Folgendes:

- Ein Azure KI Foundry-Hub.
- Ein Azure KI Foundry-Projekt
- Ein bereitgestelltes Modell (z. B. GPT-4o),

### Erstellen eines Azure KI-Hubs und eines Projekts

> **Hinweis:** Wenn Sie bereits über ein Azure KI-Projekt verfügen, können Sie dieses Verfahren überspringen und Ihr vorhandenes Projekt verwenden.

Sie können ein Azure KI-Projekt manuell über das Azure AI Foundry-Portal erstellen und das in der Übung verwendete Modell bereitstellen. Sie können diesen Prozess jedoch auch mithilfe einer Vorlagenanwendung mit [Azure Developer CLI (azd)](https://aka.ms/azd) automatisieren.

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

### Einrichten Ihrer virtuellen Umgebung in Cloud Shell

Zum schnellen Experimentieren und Durchlaufen verwenden Sie eine Reihe von Python-Skripts in Cloud Shell.

1. Geben Sie im Befehlszeilenbereich von Cloud Shell den folgenden Befehl ein, um zu dem Ordner mit den in dieser Übung verwendeten Codedateien zu navigieren:

     ```powershell
    cd ~/mslearn-genaiops/Files/03/
     ```

1. Geben Sie die folgenden Befehle ein, um eine virtuelle Umgebung zu aktivieren und die benötigten Bibliotheken zu installieren:

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv openai tiktoken azure-ai-projects prompty[azure]
    ```

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu öffnen:

    ```powershell
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei die Platzhalter **ENDPOINTNAME** und **APIKEY** durch den Endpunkt und die Schlüsselwerte, die Sie zuvor kopiert haben.
1. *Nachdem* Sie die Platzhalter ersetzt haben, verwenden Sie im Code-Editor den Befehl **STRG+S**, oder **klicken Sie mit der rechten Maustaste und klicken dann auf „Speichern“**, um Ihre Änderungen zu speichern. Verwenden Sie dann den Befehl **STRG+Q**, oder **klicken Sie mit der rechten Maustaste und klicken dann auf „Beenden“**, um den Code-Editor zu schließen, während die Cloud Shell-Befehlszeile geöffnet bleibt.

## Optimieren von System-Prompts

Das Minimieren der Länge von System-Prompts bei gleichzeitiger Aufrechterhaltung der Funktionalität in der generativen KI ist für umfangreiche Bereitstellungen von grundlegender Bedeutung. Kürzere Prompts können zu schnelleren Antwortzeiten führen, da das KI-Modell weniger Token verarbeitet und außerdem weniger Rechenressourcen verwendet.

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Anwendungsdatei zu öffnen:

    ```powershell
   code optimize-prompt.py
    ```

    Überprüfen Sie den Code, und beachten Sie, dass das Skript die Vorlagendatei `start.prompty` ausführt, die bereits über einen vordefinierten System-Prompt verfügt.

1. Führen Sie `code start.prompty` aus, um den System-Prompt zu überprüfen. Überlegen Sie, wie Sie ihn kürzen können und gleichzeitig die Absicht klar und effektiv ausdrücken. Zum Beispiel:

   ```python
   original_prompt = "You are a helpful assistant. Your job is to answer questions and provide information to users in a concise and accurate manner."
   optimized_prompt = "You are a helpful assistant. Answer questions concisely and accurately."
   ```

   Entfernen Sie redundante Wörter und konzentrieren Sie sich auf die wesentlichen Anweisungen. Speichern Sie den optimierten Prompt in der Datei.

### Testen und Überprüfen der Optimierung

Das Testen von Prompt-Änderungen ist wichtig, um sicherzustellen, dass Sie die Tokennutzung ohne Verlust der Qualität reduzieren.

1. Führen Sie `code token-count.py` aus, um die in der Übung bereitgestellte Tokenzähler-App zu öffnen und zu überprüfen. Wenn Sie einen optimierten Prompt verwendet haben, der sich von dem im obigen Beispiel unterscheidet, können Sie ihn auch in dieser App verwenden.

1. Führen Sie das Skript mit `python token-count.py` aus, und beobachten Sie den Unterschied in der Tokenanzahl. Stellen Sie sicher, dass der optimierte Prompt weiterhin qualitativ hochwertige Antworten liefert.

## Analysieren von Benutzerinteraktionen

Wenn Sie verstehen, wie Benutzende mit Ihrer App interagieren, können Sie Muster identifizieren, die die Tokennutzung erhöhen.

1. Überprüfen Sie ein Beispieldataset mit Benutzer-Prompts:

    - **„Fasse den Inhalt von *Krieg und Frieden* zusammen.“**
    - **„Was sind einige interessante Fakten über Katzen?“**
    - **„Erstelle einen detaillierten Geschäftsplan für ein Startup, das KI verwendet, um Lieferketten zu optimieren.“**
    - **„Übersetze „Hallo, wie geht's?“ ins Französische.“**
    - **„Erkläre einem 10-jährigen Kind die Quantenverschränkung.“**
    - **„Gib mir 10 kreative Ideen für eine Sci-Fi-Kurzgeschichte.“**

    Ermitteln Sie jeweils, ob es wahrscheinlich zu einer **kurzen**, **mittellangen** oder **langen/komplexen** Antwort der KI kommt.

1. Überprüfen Sie Ihre Kategorisierungen. Welche Muster bemerken Sie? Berücksichtigen Sie dabei Folgendes:

    - Wirkt sich die **Abstraktionsebene** (z. B. kreativ im Vergleich zu sachlich) auf die Länge aus?
    - Sind **offene Prompts** in der Regel länger?
    - Wie wirkt sich die **Komplexität der Anweisung** (z. B. „erkläre, als wenn 10 wäre“) auf die Antwort aus?

1. Geben Sie den folgenden Befehl ein, um die Anwendung **optimize-prompt** auszuführen:

    ```
   python optimize-prompt.py
    ```

1. Verwenden Sie einige der oben angegebenen Beispiele, um Ihre Analyse zu überprüfen.
1. Verwenden Sie nun den folgenden langen Prompt und überprüfen Sie die Ausgabe:

    ```
   Write a comprehensive overview of the history of artificial intelligence, including key milestones, major contributors, and the evolution of machine learning techniques from the 1950s to today.
    ```

1. Schreiben Sie diesen Prompt um, um:

    - Den Geltungsbereich zu begrenzen
    - Erwartungen an Kürze festzulegen
    - Formatierung oder Struktur zu verwenden, um Antworten zu lenken

1. Vergleichen Sie die Antworten, um zu überprüfen, ob Sie eine präzisere Antwort erhalten haben.

> **HINWEIS:** Sie können `token-count.py` verwenden, um die Tokenverwendung in beiden Antworten vergleichen.
<br>
<details>
<summary><b>Beispiel für einen umgeschriebenen Prompt:</b></summary><br>
<p>„Erstelle eine Zusammenfassung mit Aufzählungspunkten von 5 wichtigen Meilensteinen der Entwicklung der KI.“</p>
</details>

## [**OPTIONAL**] Anwenden Ihrer Optimierungen in einem echten Szenario

1. Stellen Sie sich vor, Sie erstellen einen Support-Chatbot, der schnelle und genaue Antworten liefern muss.
1. Integrieren Sie Ihren optimierte System-Prompt und die Vorlage in den Code des Chatbots (*Sie können `optimize-prompt.py` als Ausgangspunkt verwenden*).
1. Testen Sie den Chatbot mit verschiedenen Benutzerabfragen, um sicherzustellen, dass er effizient und effektiv antwortet.

## Zusammenfassung

Die Prompt-Optimierung ist eine wichtige Fähigkeit, um Kosten zu senken und die Leistung in Anwendungen mit generativer KI zu verbessern. Indem Sie Prompts verkürzen, Vorlagen verwenden und Benutzerinteraktionen analysieren, können Sie effizientere und skalierbare Lösungen erstellen.

## Bereinigen

Wenn Sie mit der Erkundung von Azure KI Services fertig sind, sollten Sie die in dieser Übung erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zur Browserregisterkarte mit dem Azure-Portal zurück (oder öffnen Sie das [Azure-Portal](https://portal.azure.com?azure-portal=true) auf einer neuen Browserregisterkarte erneut), und zeigen Sie den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.
1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.
1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
