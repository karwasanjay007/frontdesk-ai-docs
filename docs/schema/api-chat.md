# api/chat: the server function contract

`api/chat.js` is the only server code in FrontDesk AI. It asks a language model to reword an answer that
the rules engine in the browser has already worked out from the clinic file. It never decides what the
answer is: the browser sends the draft and the facts, and checks whatever comes back with `guard()` (see
[../data-model.md](../data-model.md), "The guard").

## Environment

| Variable | Use |
| --- | --- |
| `OPENROUTER_API_KEY` | The OpenRouter key, read from the server's environment on each request. Without it every POST gets 503. |
| `OPENROUTER_MODEL` | The model id. When not set, `deepseek/deepseek-v4.1-flash`. |

The key is sent only in the `Authorization: Bearer` header of the call to OpenRouter. It is never returned
in a response and never logged: the function has no logging at all.

## GET /api/chat (warm-up)

The chat page sends one GET when it opens, so that the first real question is not slowed by the function
starting up. It reads the body and ignores the result.

- 200 `{"ok": true}`. No key is needed and nothing else happens.

## POST /api/chat

Request body, JSON (`Content-Type: application/json`). The chat page sends it only for a reply of kind
`answer` that the rules engine marked as fit for the model (not greetings or thanks).

| Field | Type | Limit applied by the function |
| --- | --- | --- |
| `question` | string | Converted to a string and cut to 500 characters. Required (not empty). |
| `draft` | string | The rules engine's reply. Cut to 1,500 characters. Required (not empty). |
| `facts` | array of strings | Only when it is an array: first 20 items, each converted to a string. Otherwise none. |
| `lang` | `en`, `hi` or `te` | Any other value becomes `en`. |
| `clinic` | object | The clinic file in use. Only when it is an object: turned into JSON and cut to 12,000 characters. Otherwise empty. |

When the platform has not already parsed the body, the function reads the stream itself and stops with
400 when it passes 60,000 characters.

### What it sends to the model

One call to `https://openrouter.ai/api/v1/chat/completions` with `temperature` 0.2, `max_tokens` 700 and
reasoning turned off. The system message tells the model to rewrite the draft warmly in at most three
sentences in the chosen language (English, Hindi in Devanagari, Telugu in Telugu script), to use only
facts from the draft and the clinic file, to copy every price, time, date, doctor name and phone number
exactly, to keep the listed facts word for word, and never to give medical advice, name a medicine or
mention a discount or percentage. The user message holds the question, the draft and the clinic file.

### Time limit

The model call is cut off after 5,000 ms (an `AbortController`). The chat page waits at most 6,000 ms for
the whole request. Either way, the patient then sees the rules engine's own text.

### Responses

Every response is JSON with `Content-Type: application/json; charset=utf-8` and `Cache-Control: no-store`.

| Code | Body | When |
| --- | --- | --- |
| 200 | `{"text": "<reply>", "model": "<model id>"}` | The model answered in full. `text` is trimmed. |
| 400 | `{"error": "bad request"}` | The body is not valid JSON, or the streamed body passed 60,000 characters. |
| 400 | `{"error": "missing question or draft"}` | `question` or `draft` is empty after conversion. |
| 405 | `{"error": "POST only"}` | Any method other than GET or POST. |
| 502 | `{"error": "model <status>"}` | OpenRouter answered with a status that is not 2xx, for example `model 429`. |
| 502 | `{"error": "empty"}` | The model returned no text. |
| 502 | `{"error": "cut off"}` | The model stopped because it reached `max_tokens` (`finish_reason` is `length`). |
| 503 | `{"error": "no key"}` | `OPENROUTER_API_KEY` is not set. This is checked before the body is read. |
| 504 | `{"error": "timeout"}` | The 5,000 ms limit was reached. |
| 504 | `{"error": "failed"}` | Any other error while calling the model. |

The chat page treats every response other than a 200 with a `text` that passes `guard()` the same way: it
shows the rules engine's text and stores `source: "file"` on the ticket. The UI label "Answered from the
clinic file" is added to every reply the model did not word. Fixed
emergency, refuse, medical, booking and not-sure replies never call this function but carry the label
too.
