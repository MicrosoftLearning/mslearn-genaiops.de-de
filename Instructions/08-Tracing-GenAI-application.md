---
lab:
  title: Analysieren und Debuggen Ihrer generativen KI-App mit Ablaufverfolgung
  description: 'Hier erfahren Sie, wie Sie Ihre Anwendung für generative KI debuggen, indem Sie den Workflow von der Benutzereingabe bis hin zur Modellantwort und Nachverarbeitung nachverfolgen.'
---

# Analysieren und Debuggen Ihrer generativen KI-App mit Ablaufverfolgung

Diese Übung dauert ca. **30** Minuten.

> **Hinweis**: Diese Übung setzt gewisse Kenntnisse von Azure KI Foundry voraus. Aus diesem Grund sind einige Anweisungen bewusst weniger detailliert gehalten, um eine aktivere Erkundung und praktisches Lernen zu fördern.

## Einführung

In dieser Übung führen Sie einen mehrstufigen generativen KI-Assistenten aus, der Wandertouren empfiehlt und Outdoor-Ausrüstung vorschlägt. Sie werden die Tracing-Funktionen des Azure KI Inference SDK nutzen, um zu analysieren, wie Ihre Anwendung ausgeführt wird, und um wichtige Entscheidungspunkte zu identifizieren, die vom Modell und der umgebenden Logik getroffen werden.

Sie werden mit einem bereitgestellten Modell interagieren, um eine echte User Journey zu simulieren, jede Phase der Anwendung von der Benutzereingabe über die Modellreaktion bis hin zur Nachbearbeitung verfolgen und die Ablaufverfolgungsdaten in Azure AI Foundry anzeigen. Dies wird Ihnen helfen zu verstehen, wie Ablaufverfolgung den Einblick verbessert, das Debuggen vereinfacht und die Leistungsoptimierung von generativen KI-Anwendungen unterstützt.

## Einrichten der Umgebung

Um diese Übung abzuschließen benötigen Sie Folgendes:

- Ein Azure KI Foundry-Hub.
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

Verbinden Sie Application Insights mit Ihrem Projekt in Azure KI Foundry, um gesammelte Daten für die Analyse zu starten.

1. Verwenden Sie das Menü auf der linken Seite, und wählen Sie die Seite **Ablaufverfolgung** aus.
1. **Erstellen Sie eine neue** Application Insights-Ressource, um eine Verbindung mit Ihrer App herzustellen.
1. Geben Sie einen Application Insights-Ressourcennamen ein, und wählen Sie **Erstellen** aus.

Application Insights ist jetzt mit Ihrem Projekt verbunden, und die Daten werden für die Analyse erfasst.

## Ausführen einer generativen KI-App mit der Cloud Shell

Sie stellen eine Verbindung zu Ihrem Azure AI Foundry-Projekt über Azure Cloud Shell her und interagieren programmatisch mit einem bereitgestellten Modell als Teil einer generativen KI-Anwendung.

### Interagieren mit einem bereitgestellten Modell

Beginnen Sie mit dem Abrufen der notwendigen Informationen, um sich für die Interaktion mit Ihrem bereitgestellten Modell zu authentifizieren. Anschließend greifen Sie auf die Azure Cloud Shell zu und aktualisieren den Code Ihrer generativen KI-App.

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
   cd mslearn-genaiops/Files/08
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

### Aktualisieren des Codes für Ihre generative KI-App

Nun, da Ihre Umgebung eingerichtet und Ihre .env-Datei konfiguriert ist, ist es an der Zeit, Ihr KI-Assistentenskript für die Ausführung vorzubereiten. Neben der Verbindung mit einem KI-Projekt und der Aktivierung von Application Insights müssen Sie Folgendes ausführen:

- Interagieren Sie mit Ihrem bereitgestellten Modell.
- Definieren Sie die Funktion, um Ihre Eingabeaufforderung anzugeben.
- Definieren Sie den Hauptfluss, der alle Funktionen aufruft.

Sie fügen diese drei Teile zu einem Startskript hinzu.

1. Führen Sie den folgenden Befehl aus, um **das bereitgestellte Skript zu öffnen**:

    ```
   code start-prompt.py
    ```

    Sie werden sehen, dass mehrere wichtige Zeilen leer gelassen oder mit leeren # Kommentaren versehen wurden. Ihre Aufgabe besteht darin, das Skript durch Kopieren und Einfügen der korrekten Zeilen an die entsprechenden Speicherorte abzuschließen.

1. Suchen Sie im Skript die **# Funktion zum Aufrufen des Modells und zur Behandlung der Ablaufverfolgung**.
1. Unter diesem Kommentar fügen Sie den folgenden Code ein:

    ```
   # Function to call the model and handle tracing
   def call_model(system_prompt, user_prompt, span_name):
       with tracer.start_as_current_span(span_name) as span:
           span.set_attribute("session.id", SESSION_ID)
           span.set_attribute("prompt.user", user_prompt)
           start_time = time.time()
    
           response = chat_client.chat.completions.create(
               model=model_deployment,
               messages=[
                   { 
                       "role": "system", 
                       "content": system_prompt 
                   },
                   { 
                       "role": "user", 
                       "content": user_prompt
                   }
               ]
           )
    
           duration = time.time() - start_time
           output = response.choices[0].message.content
           span.set_attribute("response.time", duration)
           span.set_attribute("response.tokens", len(output.split()))
           return output
    ```

1. Suchen Sie im Skript die **# Funktion zur Empfehlung einer Wanderung auf der Grundlage von Benutzerpräferenzen**.
1. Unter diesem Kommentar fügen Sie den folgenden Code ein:

    ```
   # Function to recommend a hike based on user preferences 
   def recommend_hike(preferences):
        with tracer.start_as_current_span("recommend_hike") as span:
            prompt = f"""
            Recommend a named hiking trail based on the following user preferences.
            Provide only the name of the trail and a one-sentence summary.
            Preferences: {preferences}
            """
            response = call_model(
                "You are an expert hiking trail recommender.",
                prompt,
                "recommend_model_call"
            )
            span.set_attribute("hike_recommendation", response.strip())
            return response.strip()
    ```

1. Suchen Sie in dem Skript nach **# ---- Main Flow ----**.
1. Unter diesem Kommentar fügen Sie den folgenden Code ein:

    ```
   if __name__ == "__main__":
       with tracer.start_as_current_span("trail_guide_session") as session_span:
           session_span.set_attribute("session.id", SESSION_ID)
           print("\n--- Trail Guide AI Assistant ---")
           preferences = input("Tell me what kind of hike you're looking for (location, difficulty, scenery):\n> ")

           hike = recommend_hike(preferences)
           print(f"\n✅ Recommended Hike: {hike}")

           # Run profile function


           # Run match product function


           print(f"\n🔍 Trace ID available in Application Insights for session: {SESSION_ID}")
    ```

1. **Speichern Sie die Änderungen**, die Sie im Skript vorgenommen haben.
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

1. Geben Sie eine Beschreibung der Art der Wanderung, die Sie suchen, z. B.:

    ```
   A one-day hike in the mountains
    ```

    Das Modell erzeugt eine Antwort, die mit Application Insights erfasst wird. Sie können die Ablaufverfolgungen im **Azure AI Foundry-Portal** visualisieren.

> **Hinweis**: Es kann einige Minuten dauern, bis Überwachungsdaten in Azure Monitor angezeigt werden.

## Anzeigen Ihrer Ablaufverfolgungsdaten im Azure AI Foundry-Portal

Nachdem Sie das Skript ausgeführt haben, haben Sie ein Protokoll der Ausführung Ihrer KI-Anwendung erstellt. Jetzt erkunden Sie es mithilfe von Application Insights in Azure AI Foundry.

> **Hinweis:** Später führen Sie den Code erneut aus und zeigen die Ablaufverfolgungen erneut im Azure KI Foundry-Portal an. Sehen wir uns zunächst an, wo die Ablaufverfolgungen gefunden werden sollen, um sie zu visualisieren.

### Navigieren Sie zum Azure AI Foundry-Portal

1. **Lassen Sie Cloud Shell geöffnet!** Sie werden darauf zurückkommen, um den Code zu aktualisieren und ihn erneut auszuführen.
1. Navigieren Sie in Ihrem Browser zu der Registerkarte, auf der das **Azure AI Foundry-Portal** geöffnet ist.
1. Wählen Sie auf der linken Seite das Menü **Ablaufverfolgung** aus.
1. *Wenn* keine Daten angezeigt werden, **aktualisieren** Sie Ihre Ansicht.
1. Wählen Sie die Ablaufverfolgung **train_guide_session** aus, um ein neues Fenster zu öffnen, in dem weitere Details angezeigt werden.

### Überprüfen Ihrer Ablaufverfolgung

Diese Ansicht zeigt die Ablaufverfolgung für eine vollständige Sitzung des Trail Guide AI-Assistenten.

- **Übergeordnete Spanne**: trail_guide_session (die übergeordnete Spanne). Sie stellt die gesamte Ausführung Ihres Assistenten von Anfang bis Ende dar.

- **Verschachtelte untergeordnete Spannen**: Jede eingerückte Linie stellt einen geschachtelten Vorgang dar. Sie finden Folgendes:

    - **recommend_hike**: Erfasst Ihre Logik bei der Entscheidung einer Wanderung.
    - **recommend_model_call**: Ist die von call_model() innerhalb von recommend_hike erstellte Spanne.
    - **chat gpt-4o**: Dies wird automatisch vom Azure AI Inference SDK instrumentiert, um die tatsächliche LLM-Interaktion anzuzeigen.

1. Sie können auf eine beliebige Spanne klicken, um Folgendes anzuzeigen:

    1. Die Dauer.
    1. Die Attribute wie Benutzerprompt, verwendete Tokens, Antwortzeit.
    1. Alle Fehler oder benutzerdefinierten Daten, die mit **span.set_attribute(...)** verbunden sind.

## Hinzufügen weiterer Funktionen zum Code

1. Navigieren Sie in Ihrem Browser zu der Registerkarte, auf der das **Azure-Portal** geöffnet ist.
1. Führen Sie den folgenden Befehl aus, um **das Skript erneut zu öffnen:**

    ```
   code start-prompt.py
    ```

1. Suchen Sie im Skript **# Function to generate a trip profile for the recommended hike**.
1. Unter diesem Kommentar fügen Sie den folgenden Code ein:

    ```
   def generate_trip_profile(hike_name):
       with tracer.start_as_current_span("trip_profile_generation") as span:
           prompt = f"""
           Hike: {hike_name}
           Respond ONLY with a valid JSON object and nothing else.
           Do not include any intro text, commentary, or markdown formatting.
           Format: {{ "trailType": ..., "typicalWeather": ..., "recommendedGear": [ ... ] }}
           """
           response = call_model(
               "You are an AI assistant that returns structured hiking trip data in JSON format.",
               prompt,
               "trip_profile_model_call"
           )
           print("🔍 Raw model response:", response)
           try:
               profile = json.loads(response)
               span.set_attribute("profile.success", True)
               return profile
           except json.JSONDecodeError as e:
               print("❌ JSON decode error:", e)
               span.set_attribute("profile.success", False)
               return {}
    ```

1. Suchen Sie im Skript **# Function to match recommended gear with products in the catalog**.
1. Unter diesem Kommentar fügen Sie den folgenden Code ein:

    ```
   def match_products(recommended_gear):
       with tracer.start_as_current_span("product_matching") as span:
           matched = []
           for gear_item in recommended_gear:
               for product in mock_product_catalog:
                   if any(word in product.lower() for word in gear_item.lower().split()):
                       matched.append(product)
                       break
           span.set_attribute("matched.count", len(matched))
           return matched
    ```

1. Suchen Sie im Skript **# Run profile function**.
1. Fügen Sie unterhalb und **ausgerichtet mit** diesem Kommentar den folgenden Code ein:

    ```
           profile = generate_trip_profile(hike)
           if not profile:
               print("Failed to generate trip profile. Please check Application Insights for trace.")
               exit(1)

           print(f"\n📋 Trip Profile for {hike}:")
           print(json.dumps(profile, indent=2))
    ```

1. Suchen Sie im Skript **# Run match product function**.
1. Fügen Sie unterhalb und **ausgerichtet mit** diesem Kommentar den folgenden Code ein:

    ```
           matched = match_products(profile.get("recommendedGear", []))
           print("\n🛒 Recommended Products from Lakeshore Retail:")
           print("\n".join(matched))
    ```

1. **Speichern Sie die Änderungen**, die Sie im Skript vorgenommen haben.
1. Geben Sie im Cloud Shell-Befehlszeilenfenster unterhalb des Code-Editors den folgenden Befehl ein, **um das Skript auszuführen**:

    ```
   python start-prompt.py
    ```

1. Beschreiben Sie bitte die Art der Wanderung, die Sie suchen, zum Beispiel:

    ```
   I want to go for a multi-day adventure along the beach
    ```

<br>
<details>
<summary><b>Lösungsskript:</b> Falls Ihr Code nicht funktioniert.</summary><br>
<p>Wenn Sie die LLM-Ablaufverfolgung für die generate_trip_profile-Funktion prüfen, werden Sie feststellen, dass die Antwort des Assistenten Backticks und das Wort JSON enthält, um die Ausgabe als Codeblock zu formatieren.

Dies ist zwar hilfreich für die Anzeige, verursacht jedoch Probleme im Code, da die Ausgabe nicht mehr gültig ist. Dies führt zu einem Analysefehler während der weiteren Verarbeitung.

Der Fehler ist wahrscheinlich darauf zurückzuführen, dass der LLM angewiesen ist, ein bestimmtes Format für seine Ausgabe einzuhalten. Die Aufnahme der Anweisungen in die Benutzeraufforderung scheint effektiver zu sein als die Aufnahme in die Systemaufforderung.</p>
</details>


> **Hinweis**: Es kann einige Minuten dauern, bis Überwachungsdaten in Azure Monitor angezeigt werden.

### Anzeigen Ihrer Ablaufverfolgungen im Azure AI Foundry-Portal

1. Navigieren Sie zum Azure KI Foundry-Portal.
1. Eine neue Ablaufverfolgung mit demselben Namen **trail_guide_session** sollte angezeigt werden. Aktualisieren Sie ihre Ansicht bei Bedarf.
1. Wählen Sie die neue Ablaufverfolgung aus, um die detailliertere Ansicht zu öffnen.
1. Überprüfen Sie die neuen geschachtelten untergeordneten Abschnitte **trip_profile_generation** und **product_matching**.
1. Wählen Sie **product_matching** aus, und überprüfen Sie die angezeigten Metadaten.

    In der Funktion „product_matching“ haben Sie **span.set_attribute("matched.count", len(matched))** eingefügt. Durch Festlegen des Attributs mit dem Schlüssel-Wert-Paar **matched.count** und der Länge der Variablen „matched“ haben Sie diese Informationen zur Ablaufverfolgung **product_matching** hinzugefügt. Sie finden dieses Schlüssel-Wert-Paar unter **Attributen** in den Metadaten.

## (OPTIONAL) Ablaufverfolgung eines Fehlers

Wenn Sie zusätzliche Zeit haben, können Sie überprüfen, wie Ablaufverfolgungen verwendet werden, wenn ein Fehler auftritt. Ein Skript, das wahrscheinlich einen Fehler auslöst, wird Ihnen zur Verfügung gestellt. Führen Sie es aus, und überprüfen Sie die Ablaufverfolgungen.

Dies ist eine Übung, die Sie herausfordern soll, daher sind die Anweisungen bewusst weniger detailliert gehalten.

1. Öffnen Sie in der Cloud Shell das Skript **error-prompt.py**. Dieses Skript befindet sich im selben Verzeichnis wie das Skript **start-prompt.py**. Überprüfen Sie den Inhalt.
1. Führen Sie das Skript **error-prompt.py** aus. Geben Sie eine Antwort in der Befehlszeile an, wenn Sie dazu aufgefordert werden.
1. *Hoffentlich* enthält die Ausgabemeldung den folgenden Text: **Die Reiseerstellung ist fehlgeschlagen. Überprüfen Sie Application Insights auf eine Ablaufverfolgung.**.
1. Navigieren Sie zur Ablaufverfolgung für die **trip_profile_generation** und untersuchen Sie, warum ein Fehler aufgetreten ist.

<br>
<details>
<summary><b>Erhalten Sie die Antwort auf</b>: Warum bei Ihnen ein Fehler aufgetreten sein könnte...</summary><br>
<p>Wenn Sie die LLM-Ablaufverfolgung für die generate_trip_profile-Funktion prüfen, werden Sie feststellen, dass die Antwort des Assistenten Backticks und das Wort JSON enthält, um die Ausgabe als Codeblock zu formatieren.

Dies ist zwar hilfreich für die Anzeige, verursacht jedoch Probleme im Code, da die Ausgabe nicht mehr gültig ist. Dies führt zu einem Analysefehler während der weiteren Verarbeitung.

Der Fehler ist wahrscheinlich darauf zurückzuführen, dass der LLM angewiesen ist, ein bestimmtes Format für seine Ausgabe einzuhalten. Die Aufnahme der Anweisungen in die Benutzeraufforderung scheint effektiver zu sein als die Aufnahme in die Systemaufforderung.</p>
</details>

## Wo finde ich andere Übungsszenarien?

Weitere Übungen und Übungsaufgaben finden Sie im [Azure KI-Foundry-Lernportal](https://ai.azure.com) oder im **Übungsteil** des Kurses.
