# [Google Summer of Code][gsoc] 2024, Project Report

- **Project Title:** Creating an HTTP Client using Dart Bindings from OkHttp

- **Organization:** Dart

- **Contributor:** [Anikate De][me]

- **Mentors:** [Brian Quinlan][brianquinlan], [Camille Simon][camsim99]

- **Project Size:** Large, 350 hours

## Project Description

This project aimed at creating a novel Flutter plugin that integrates the [OkHttp][okhttp] library with [Dart][], to provide an HTTP Client for [Flutter][flutter] applications. This integration would be facilitated using Dart's [JNIgen][jnigen] package, which generates JNI bindings for Java & Kotlin code. Since OkHttp can interact with native Android APIs, this plugin was expected to provide support for the following user-requested features:

1. Support for `KeyStore` `PrivateKey`s ([#50669][50669])

2. Support for the System Proxy ([#50434][50434])

3. Support for User-Installed Certificates ([#50435][50435])

The aforementioned Client was expected to be conformant with the other HTTP Clients available in the [Dart HTTP mono-repo][dart http], to ensure a seamless transition for the users.

Finally, the project also aimed at creating a new [WebSocketChannel][wschannel] using OkHttp's WebSocket APIs.

## Project Milestones

During my initial proposal, the project was broadly divided into four milestones:

1. **Create a bare-bones `package: ok_http`:** Using an FFI package template to create a new package, and setting up the necessary dependencies.

2. **Implement the BaseClient Interface and execute requests synchronously:** Not using asynchronous APIs from OkHttp yet, this milestone ensures that the Client was able to execute basic requests.

3. **Execute Asynchronous Requests and Stream ResponseBody**: Using OkHttp's `enqueue()` API, this milestone aimed at executing asynchronous requests and streaming the response body. Due to the nature of `InputStream`s in Java, several challenges were expected to be faced.

4. **WebSocket Implementation**: The final milestone proposed the creation of a new `OkHttpWebSocket` Class using OkHttp's `WebSocket` and `WebSocketListener` APIs.

## Overall Project Flow

From the above milestones, the following project flow was expected:

1. Set up the package and dependencies.

2. Identify and generate the JNI Bindings of the required OkHttp APIs using `package: jnigen`.

3. Implement the `BaseClient` interface and execute synchronous requests.

4. Implement the `enqueue()` API and execute asynchronous requests.

5. Stream the response body.

6. Pass all conformance tests in `package: http_client_conformance_tests`.

7. Implement the `WebSocket` interface from `package: web_socket`.

8. Pass all conformance tests in `package: web_socket_conformance_tests`.

9. Publish the package to [pub.dev][pub]!

<details>
    <summary>
        Spoiler Alert!
    </summary>

    All the above steps were completed successfully and on schedule! The package was published to pub.dev on the 5th of August, 2024.

</details>

## Contributions

### Pull Requests

Starting May 29th, 2024, I made the following Pull Requests to the [Dart HTTP mono-repo][dart http]:

| # | Title | Status |
| --- | --- | --- |
| 1 | [`ok_http`: Add BaseClient Implementation and make asynchronous requests. #1215][1215] | Merged :white_check_mark: |
| 2 | [pkgs/ok_http: Condense JNI Bindings to `single_file` structure, and add missing server errors test #1221][1221] | Merged :white_check_mark: |
| 3 | [pkgs/http_client_conformance_tests: add boolean flag `supportsFoldedHeaders` #1222][1222] | Merged :white_check_mark: |
| 4 | [pkgs/ok_http: Close and remove all idle connections from the resource pool on response #1223][1223] | Closed :x: |
| 5 | [pkgs/ok_http: Close the client (override `close()` from `BaseClient`) #1224][1224] | Merged :white_check_mark: |
| 6 | [http_client_conformance_tests: Fix inconsistent test server behavior #1227][1227] | Merged :white_check_mark: |
| 7 | [\[ok_http\]: Use the Android SDK to generate JNI bindings. #1229][1229] | Merged :white_check_mark: |
| 8 | [[pkgs/ok_http] Add functionality to accept and configure redirects. #1230][1230] | Merged :white_check_mark: |
| 9 | [pkgs/ok_http: Stream response bodies. #1233][1233] | Merged :white_check_mark: |
| 10 | [pkgs/ok_http: DevTools Networking Support. #1242][1242] | Merged :white_check_mark: |
| 11 | [pkgs/ok_http: JNIgen fixes and added WebSocket support #1257][1257] | Merged :white_check_mark: |
| 12 | [[docs] Add ok_http entry to README.md #1285][1285] | Merged :white_check_mark: |
| 13 | [[docs] Sort pkg list in lexical order. #1287][1287] | Merged :white_check_mark: |
| 14 | [pkgs/ok_http: OkHttpClientConfiguration and configurable timeouts. #1289][1289] | Merged :white_check_mark: |

Additionally, I created another PR in the [Dart Native][dart native] repository:

| # | Title | Status |
| --- | --- | --- |
| 1 | [pkgs/jni: Support Throwing Java Exceptions in Interface Impl. #1211][1211] | Merged :white_check_mark: |

### Issues

| # | Repository | Title | Status |
| --- | --- | --- | --- |
| 1 | [dart-lang/http][dart http] | [[http_client_conformance_tests] Add flag to skip tests for folded headers #1219][1219] | Closed :white_check_mark: |
| 2 | [dart-lang/http][dart http] | [\[ok_http\] Support Android Keystore PrivateKeys #1237][1237] | Open :warning: |
| 3 | [dart-lang/native][dart native] | [[jni] Deadlock when calling `Interceptor.Chain.proceed()` via bindings of Kotlin Package: `OkHttp` #1337][1337] | Open :warning: |

## Outcome

After around 75 days of consistent contributions and development (excluding the sample project development period), `package: ok_http` was released on August 5th, 2024. The package was published to [pub.dev][pub], and is now available for use by the Flutter Community. :partying_face::tada:

Click here to see [ok_http ![pub package](https://img.shields.io/pub/v/ok_http.svg)][ok_http]

All the milestones were adequately completed, and the package passed all the HTTP Client and WebSocket conformance tests in the [Dart HTTP mono-repo][dart http]. The package was also documented thoroughly, and extra features, such as support for Flutter DevTools Network Tab, and configurable timeouts, were also added.

Interestingly, using the package leads to a valuable reduction in the APK size of Flutter applications, adding to its benefits.

`package: ok_http` is now a part of the Dart HTTP mono-repo, and is expected to be maintained by both myself and the Dart Team.

## Challenges

Although JNIgen is a powerful tool, it is an experimental package. An array of issues piled up while generating bindings for OkHttp, especially while making `package: ok_http` conformant.

The first issue, whose severity went unnoticed, was due to threading model of JNI.

### [Deadlock when testing multiple clients][multiple clients deadlock]

Due to the architecture of the integration tests, the OkHttpClient was forced to wait for a response from the server. Although insignificant visually, this led to a deadlock, eventually leading to a `java.net.SocketTimeoutException`.

As predicted in the comment, this issue was resolved after adding support for Streamed Responses, as the `InputStream` was read asynchronously, and did not block the main thread.

### OkHttp's Hard Limit on the Number of Redirects

OkHttp has a hard limit of 20 redirects, which is not configurable. This was a major issue, as the conformance tests expected the client to follow the redirects, that are customized by the user.

This required the implementation of a custom `Interceptor` that would keep track of the number of redirects, and throw an exception if the limit was reached.

This simple solution, however, turned into a prolonged issue, as the `Interceptor` is a "special" interface in its own right. `Interceptor.Chain.proceed()` led to deadlocks and made it impossible for Dart JNI Bindings to implement them.

A workaround was found, dedicated Kotlin code was written to handle the redirects, and the `Interceptor` was implemented in Kotlin, instead of Dart.

[The issue is still open][1337], and is expected to be resolved in the future.

### Streaming Response Bodies

Streaming the response bodies was a major challenge, as the `InputStream` in Java is blocking, and does not support asynchronous reading. This required the use of a separate thread to read the `InputStream`, and pass the data to the Dart side.

Again, the threading model of JNI came into play, forcing the use of dedicated Kotlin code to handle the streaming in a multi-threaded manner.

### Shortcomings of a few conformance tests

The conformance tests in the Dart HTTP mono-repo are quite strict, and expect the client to behave in a certain way. This led to a few issues, especially with supporting folded headers. A new flag was added to skip these tests, as the client was not expected to support them. You can read more about this issue [here][1219].

Furthermore, `testRequestCookies` was deemed as "racy" by Brian which was fixed in [this PR][1227].

### "Why should Redirects have all the fun?" -WebSockets

Just like the redirects, the WebSocket conformance tests were also quite strict and expected the client to only contain a specific set of headers in responses. This required creating another implementation of `Interceptor`, called the `WebSocketInterceptor`, which would only add the required headers to the WebSocket handshake.

## Future Work & Maintenance

`package: ok_http` turned out to be a great success. Smaller APK sizes, WebSocket support, Flutter DevTools Networking Tab support, and a seamless integration with the Dart HTTP mono-repo, are some of the highlights of the project.

It has been a great journey, and I am looking forward to maintaining `package: ok_http` in the future, along with the Dart team. Some of the future work includes:

1. **Adding Response Cache Configuration:** OkHttp provides a powerful `Cache` API, which can be used to cache responses. This would be a great addition to the package.

2. **Support for other Protocols:** We wish to support a wider range of protocols supported by OkHttp, such as HTTP/2, and QUIC.

## Things I Learned

1. **Flutter Integration Testing:** I learned how to write integration tests for Flutter applications, and how to test the HTTP Client using the `package: http_client_conformance_tests`.

2. **JNI Threading Model:** A HUGE thanks to [Hossein Yousefi][HosseinYousefi] (although not the assigned mentor in this project, he went out of his way to review my PRs and schedule demo meetings, and answer all of my doubts) for helping me understand the threading model of JNI, and how to work around it.

3. **Using JNIgen:** JNIgen is an awesome tool, and I learned how to generate JNI bindings for Java and Kotlin code using it. I can see myself using it in future projects as well.

4. **Reading HTTP and WebSocket RFCs:** I read the HTTP and WebSocket RFCs to understand how the protocols work, and how the clients should behave. This was a great learning experience.

## Acknowledgements

I would like to thank the Dart Team, especially Brian and Camille, for guiding me throughout the project and helping me resolve the issues I faced. I would also like to thank Hossein for his invaluable help and support.

I would also like to thank the Flutter Community for their support and feedback, and the Google Summer of Code team for providing me with this opportunity.

This summer has been nothing short of amazing, and I am looking forward to contributing to the Dart Community, and more open-source projects in the future.

### Happy Open Sourcery! :mage::magic_wand::gift_heart:

[gsoc]: https://summerofcode.withgoogle.com/
[me]: https://github.com/Anikate-De
[brianquinlan]: https://github.com/brianquinlan
[camsim99]: https://github.com/camsim99
[HosseinYousefi]: https://github.com/HosseinYousefi
[okhttp]: https://square.github.io/okhttp/
[flutter]: https://flutter.dev/
[dart]: https://dart.dev/
[50669]: https://github.com/dart-lang/sdk/issues/50669
[50434]: https://github.com/dart-lang/sdk/issues/50434
[50435]: https://github.com/dart-lang/sdk/issues/50435
[dart http]: https://github.com/dart-lang/http
[dart native]: https://github.com/dart-lang/native
[wschannel]: https://pub.dev/documentation/web_socket_channel/latest/web_socket_channel/WebSocketChannel-class.html
[jnigen]: https://github.com/dart-lang/native/tree/main/pkgs/jnigen
[pub]: https://pub.dev/
[1215]: https://github.com/dart-lang/http/pull/1215
[1221]: https://github.com/dart-lang/http/pull/1221
[1222]: https://github.com/dart-lang/http/pull/1222
[1223]: https://github.com/dart-lang/http/pull/1223
[1224]: https://github.com/dart-lang/http/pull/1224
[1227]: https://github.com/dart-lang/http/pull/1227
[1229]: https://github.com/dart-lang/http/pull/1229
[1230]: https://github.com/dart-lang/http/pull/1230
[1233]: https://github.com/dart-lang/http/pull/1233
[1242]: https://github.com/dart-lang/http/pull/1242
[1257]: https://github.com/dart-lang/http/pull/1257
[1285]: https://github.com/dart-lang/http/pull/1285
[1287]: https://github.com/dart-lang/http/pull/1287
[1289]: https://github.com/dart-lang/http/pull/1289
[1219]: https://github.com/dart-lang/http/issues/1219
[1237]: https://github.com/dart-lang/http/issues/1237
[1211]: https://github.com/dart-lang/native/pull/1211
[1337]: https://github.com/dart-lang/native/issues/1337
[ok_http]: https://pub.dev/packages/ok_http
[multiple clients deadlock]: https://github.com/dart-lang/http/pull/1215#issuecomment-2138459377
