# Code-Erkl�rung: Career Profile Q&A Assistant

Diese Anleitung erkl�rt das Projekt in einfachen Schritten. Der Benutzer fragt das Modell zu einem Lebenslauf. Das Modell erh�lt den ausgelesenen Text als Kontext und antwortet im Terminal.

## 1. Was macht das Programm?

```text
PDF oder TXT  Text extrahieren  OpenRouter API  Antwort im Terminal
```

Das ist eine einfache LLM-Anwendung. Sie ist **kein autonomer Agent** und verwendet **kein RAG**: Das Programm sucht nicht gezielt in Textabschnitten, sondern sendet den Profiltext direkt als Kontext.

## 2. Pakete und Einstellungen

- `pypdf` liest Text aus PDF-Dateien.
- `openai` stellt den Chat-Completions-Aufruf bereit.
- `python-dotenv` l�dt die lokale `.env`-Datei in die Umgebungsvariablen.
- `argparse`, `os` und `pathlib` geh�ren zu Python und verwalten Argumente, Einstellungen und Dateipfade.

```python
from dotenv import load_dotenv
from openai import APIError, OpenAI
from pypdf import PdfReader
```

`OPENROUTER_API_KEY` und `OPENROUTER_MODEL` stehen in `.env`. Die Datei `.env` wird durch `.gitignore` vom Git-Upload ausgeschlossen. `.env.example` enth�lt nur Platzhalter; dort geh�rt kein echter Schl�ssel hinein.

## 3. Profiltext aus der Datei lesen

```python
def load_profile(path: Path) -> str:
```

Die Funktion pr�ft die Dateiendung. Bei PDF-Dateien liest `PdfReader` jede Seite und verbindet den extrahierten Text. Bei `.txt` liest Python die Datei als UTF-8. Ein leerer oder nicht unterst�tzter Inhalt f�hrt zu einer verst�ndlichen Fehlermeldung.

Das Zeichenlimit von 30.000 Zeichen ist eine einfache Schutzma�nahme gegen sehr gro�e Eingaben. Es garantiert nicht, dass jedes Modell diese Eingabe akzeptiert: Modelle haben unterschiedliche Kontextgrenzen.

Ein gescanntes PDF ist oft nur ein Bild. `pypdf` f�hrt keine OCR durch; f�r solche Dateien w�re ein separater OCR-Schritt n�tig.

## 4. Systemnachricht bauen

```python
def make_system_message(profile_text: str) -> str:
```

Die `system`-Nachricht enth�lt zwei Dinge:

1. Regeln: nur belegte Angaben verwenden und fehlende Informationen offen benennen.
2. Den extrahierten Profiltext als Kontext.

Der Hinweis, Anweisungen im Dokument als Dokumenttext zu behandeln, ist eine sinnvolle Vorsichtsma�nahme gegen Prompt Injection. Er ist keine Garantie, dass ein Modell jeden Angriff abwehrt.

## 5. API-Schl�ssel und Modell laden

```python
load_dotenv()
api_key = os.getenv("OPENROUTER_API_KEY", "").strip()
model = os.getenv("OPENROUTER_MODEL", "").strip()
```

`load_dotenv()` l�dt die lokale Konfiguration. `os.getenv()` liest Werte aus der Umgebung. Der API-Schl�ssel wird nicht in den Programmtext geschrieben und nicht ausgegeben.

Wenn ein Platzhalter stehen bleibt oder ein Wert fehlt, beendet `argparse` das Programm mit einer erkl�renden Meldung.

## 6. OpenAI-SDK mit OpenRouter verwenden

```python
client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=api_key,
)
```

Das Projekt verwendet das OpenAI-Python-SDK, richtet dessen `base_url` aber auf OpenRouter. Der Modellname kommt aus `.env`; dadurch ist er nicht im Code fest eingebaut.

## 7. Gespr�chsverlauf und Rollen

```python
messages = [{"role": "system", "content": system_message}]
```

`messages` ist die Liste, die an das Modell gesendet wird:

- `system`: Regeln und Profilkontext.
- `user`: die Frage des Benutzers.
- `assistant`: die Antwort des Modells.

Nach jeder Runde bleiben die letzten Nachrichten im Speicher. Deshalb kann der Benutzer eine Folgefrage stellen. Beim Beenden des Programms geht dieser Verlauf verloren.

## 8. Hauptschleife und Antwort

Die Schleife liest Fragen, erkennt `exit` oder `quit` und ruft dann auf:

```python
response = client.chat.completions.create(
    model=model,
    messages=messages,
)
```

Das Modell wird vom API-Anbieter ausgef�hrt. `response.choices[0].message.content` enth�lt die Antwort, die das Programm im Terminal ausgibt.

`APIError` f�ngt API-Fehler ab, damit ein falscher Modellname oder ein Verbindungsfehler nicht als langer Python-Traceback erscheint. Ein echter HTTP-Status wird angezeigt; der API-Schl�ssel wird nicht angezeigt.

## 9. Was war im urspr�nglichen Notebook problematisch?

- `google.colab.userdata` funktioniert nur in Google Colab. Dieses Projekt verwendet stattdessen `.env`.
- Die letzte Zelle war als Markdown markiert; Python darin wird nicht ausgef�hrt.
- Die Schleife war doppelt vorhanden. Eine Schleife speicherte die Antwort, zeigte sie aber nicht an.
- `Markdown` musste importiert werden, bevor `display(Markdown(...))` funktioniert.
- Das Anzeigen der ersten Zeichen eines API-Schl�ssels ist unn�tig. Der Schl�ssel sollte gar nicht ausgegeben werden.
- Das Modell wurde als fester Wert im Notebook gesetzt. Hier steht es in der lokalen Konfiguration.

## 10. Datenschutz und Grenzen

Der Profiltext und die Fragen werden an OpenRouter und den ausgew�hlten Modellanbieter gesendet. Verwende f�r Lernzwecke zuerst `sample_resume.txt`. Bevor du echte pers�nliche Daten sendest, musst du die Datenschutzbedingungen der Anbieter pr�fen und dich bewusst daf�r entscheiden.

Diese einfache Anwendung f�gt den ganzen Profiltext in den Prompt ein. RAG w�rde Dokumente zun�chst in Abschnitte teilen, passende Abschnitte abrufen und nur diese als Kontext verwenden. Dieses Projekt macht das noch nicht.

## Lernkontrolle

Beantworte vor dem n�chsten Schritt diese Frage selbst: Welche drei Nachrichtenrollen k�nnen in `messages` vorkommen, und wozu dient jede Rolle?

