# AlexaServer

A small self-hosted HTTP endpoint for Amazon Alexa custom skills, written in Java.

Instead of putting skill logic on AWS Lambda, this serves the Alexa Skills Kit request/response
protocol from a machine you control: an embedded [SparkJava](http://sparkjava.com/) server accepts
the `POST` from Alexa, deserializes the JSON body with Gson, dispatches it to a matching intent
handler and returns the serialized response.

It is a 2017 proof of concept and never went further than that: the request/response model classes
and the intent dispatch mechanism work, but only one demo intent ships with it.

## How it works

- `Startup` boots SparkJava and registers a catch-all `POST /*` route.
- `IntentHelper` scans the classpath with [Reflections](https://github.com/ronmamo/reflections) for
  classes annotated with `@Intent` and collects them at startup.
- `ResponseBuilder` maps intent names to those classes, invokes the annotated execution method via
  reflection, and serializes the resulting `AlexaResponse` back to JSON.
- The `alexa.request` / `alexa.response` packages mirror the Alexa Skills Kit JSON shapes: session,
  intent, output speech (plain text or SSML), reprompt, cards and audio player directives.

## Adding an intent

Add a class, annotate it, and it is picked up on the next start — no registration needed:

```java
@Intent(name = "test", executionMethod = "execute")
public class TestIntent {
    public AlexaResponse execute(AlexaRequest request) {
        AlexaResponse response = new AlexaResponse();
        response.response.outputSpeech.type = OutputSpeechType.PlainText;
        response.response.outputSpeech.text = "Hello from my own server";
        response.response.outputSpeech.ssml = null;
        return response;
    }
}
```

## Build and run

```bash
./gradlew fatJar
java -jar build/libs/dev.renner.alexa_server.alexa-server-all-1.0-SNAPSHOT.jar
```

SparkJava listens on port 4567. Alexa only talks to a publicly reachable HTTPS endpoint with a valid
certificate, so a reverse proxy or a tunnel in front of the server is required for real skill
testing.

Heads-up on the toolchain: this targets Java 8 with the Gradle 3.3 wrapper, and `ResponseBuilder`
imports `javafx.util.Pair`, so it builds on a JDK 8 that bundles JavaFX. Newer JDKs need those two
things replaced first.

## Tech stack

Java 8 · Gradle · SparkJava 2.6 (embedded Jetty) · Gson · Reflections

## License

MIT — see [LICENSE](LICENSE).
