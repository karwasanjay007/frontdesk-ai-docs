# The wording route: /api/chat

This page describes version 1.7.2, the version on the live link.

`api/chat.js` asks a language model to reword an answer that the rules engine in the browser has already
worked out from the clinic file. It never decides what the answer is, and it never receives the patient's
message as text for the model: the model is given the draft, the facts and the clinic file. The other routes
are listed in [the architecture](../../ARCHITECTURE.md#the-routes).

## GET /api/chat

A warm-up call made when the chat page opens. It answers 200 `{"ok": true}` and does nothing else.

## POST /api/chat

The chat page sends it only for a reply of kind `answer` that the rules engine marked as fit for the model.

| Field | Type | Limit |
| --- | --- | --- |
| `draft` | string | The rules engine's reply. Required. Over 1,500 characters is refused with 413. |
| `facts` | list of strings | The first 20 are used. |
| `lang` | `en`, `hi` or `te` | Any other value becomes `en`. |
| `kind` | string | Must be `answer`, or the request is refused with 422. |
| `clinic` | object | The clinic file in use. Required. The first 12,000 characters of its JSON go to the model. |
| `q` | string, optional | Used only by the server's own check of whether the reply may be reworded. Over 1,000 characters is refused with 413. |

A body the platform has not parsed is read as a stream and refused with 400 past 60,000 characters.

## What the server does

1. Checks the request limit: 60 POST requests a minute per caller address, or the number in
   `FRONTDESK_CHAT_PER_MINUTE`.
2. Loads `core.js` and refuses a draft that is not a file answer (an emergency, medical, refusal, booking or
   not-sure reply is never reworded).
3. Makes one call to `https://openrouter.ai/api/v1/chat/completions` with the model in `OPENROUTER_MODEL`
   (default `deepseek/deepseek-v4.1-flash`), `temperature` 0, `max_tokens` 700 and reasoning off. The model is
   told to reword the draft in at most three sentences in the chosen language.
4. Runs the guard (`FD.guard`) on the model's text against the draft and the clinic file. The browser runs the
   same guard again on what it receives.

The whole request has 8 seconds on the server; the browser waits 9 seconds. The key is read from
`OPENROUTER_API_KEY`, sent only to OpenRouter, and never returned.

## Replies

Every reply is JSON with `Cache-Control: no-store`.

| Code | Body | When |
| --- | --- | --- |
| 200 | `{"text": "...", "model": "..."}` | The model's wording passed the guard. |
| 400 | `{"error": ...}` | The body is not valid JSON, or the draft or the clinic file is missing. |
| 405 | `{"error": ...}` | A method other than GET or POST. |
| 413 | `{"error": "draft too long"}` or `"question too long"` | A field is over its limit. |
| 422 | `{"error": "not reworded"}` | The reply is not one the model may reword. |
| 422 | `{"error": "failed the check"}` | The guard rejected the model's text. |
| 429 | `{"error": ...}` | Over the request limit. |
| 502 | `{"error": ...}` | The rules engine did not load, the model returned an error, no text, or a cut-off reply. |
| 503 | `{"error": "no key"}` | `OPENROUTER_API_KEY` is not set. |
| 504 | `{"error": ...}` | The time limit was reached, or the call failed. |

On anything other than a 200 whose text passes the guard, the chat page shows the rules engine's own text and
stores `source: "file"` on the ticket.
