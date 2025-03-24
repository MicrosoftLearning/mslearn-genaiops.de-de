---
lab:
  title: Optimieren des Modells mithilfe eines synthetischen Datasets
---

## Optimieren des Modells mithilfe eines synthetischen Datasets

Das Optimieren einer generativen KI-Anwendung umfasst die Nutzung von Datasets, um die Leistung und Zuverlässigkeit des Modells zu verbessern. Durch die Verwendung synthetischer Daten können Fachkräfte in der Entwicklung eine Vielzahl von Szenarien und Grenzfällen simulieren, die in realen Daten möglicherweise nicht vorhanden sind. Darüber hinaus ist die Bewertung der Ergebnisse des Modells entscheidend, um qualitativ hochwertige und zuverlässige KI-Anwendungen zu erhalten. Der gesamte Optimierungs- und Evaluierungsprozess kann mithilfe des Azure KI-Bewertungs-SDK effizient verwaltet werden, das robuste Tools und Frameworks zur Optimierung dieser Aufgaben bereitstellt.

## Szenario

Stellen Sie sich vor, Sie möchten eine KI-gesteuerte Smart-Guide-App entwickeln, um das Erlebnis für Besuchende in einem Museum zu verbessern. Die App zielt darauf ab, Fragen zu historischen Figuren zu beantworten. Um die Antworten aus der App auszuwerten, müssen Sie ein umfassendes synthetisches Frage-Antwort-Dataset erstellen, das verschiedene Aspekte dieser Persönlichkeiten und ihrer Arbeit abdeckt.

Sie haben ein GPT-4-Modell ausgewählt, um generative Antworten bereitzustellen. Sie möchten nun einen Simulator zusammenstellen, der kontextbezogene Interaktionen generiert und die Leistung der KI in verschiedenen Szenarien auswertet.

Beginnen wir mit der Bereitstellung der erforderlichen Ressourcen zum Erstellen dieser Anwendung.

## Erstellen eines Azure KI-Hubs und eines Projekts

Sie können einen Azure KI-Hub erstellen und manuell über das Azure AI Foundry-Portal projizieren sowie die in der Übung verwendeten Modelle bereitstellen. Sie können diesen Prozess jedoch auch mithilfe einer Vorlagenanwendung mit [Azure Developer CLI (azd)](https://aka.ms/azd) automatisieren.

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
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Kopieren Sie diese Werte, da sie später verwendet werden.

## Einrichten Ihrer lokalen Entwicklungsumgebung

Um schnell zu experimentieren und zu durchlaufen, verwenden Sie ein Notizbuch mit Python-Code in Visual Studio (VS) Code. Lassen Sie uns VS Code für die lokale Ideenfindung vorbereiten.

1. Öffnen Sie den VS Code und **klonen Sie** das folgende Git-Repository: [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git)
1. Speichern Sie den Klon auf einem lokalen Laufwerk, und öffnen Sie den Ordner nach dem Klonen.
1. Öffnen Sie im VS Code Explorer (linker Bereich) das Notebook **06-Optimize-your-model.ipynb** im Ordner **Files/06**.
1. Führen Sie alle Zellen im Notebook aus.

## Bereinigen

Wenn Sie mit der Erkundung von Azure KI Services fertig sind, sollten Sie die in dieser Übung erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Kehren Sie zur Browserregisterkarte mit dem Azure-Portal zurück (oder öffnen Sie das [Azure-Portal](https://portal.azure.com?azure-portal=true) auf einer neuen Browserregisterkarte erneut), und zeigen Sie den Inhalt der Ressourcengruppe an, in der Sie die in dieser Übung verwendeten Ressourcen bereitgestellt haben.
1. Wählen Sie auf der Symbolleiste die Option **Ressourcengruppe löschen** aus.
1. Geben Sie den Namen der Ressourcengruppe ein, und bestätigen Sie, dass Sie sie löschen möchten.
