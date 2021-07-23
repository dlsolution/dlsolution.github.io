---
layout: post
title: "Automatic API request retries"
author: "Linh Vo"
tags: "Networking"
---

# NetworkClient with Retries

The following alternative service adds automatic retries to the NetworkClient service.

```swift
class NetworkClient {
    static let shared = NetworkClient()

    private init() {}

    private let session = URLSession(configuration:
                      URLSessionConfiguration.default,
                      delegate: nil, delegateQueue: nil)

    // MARK: - Fetching with retry

    func fetchWithRetry(url: String,
                   completion: @escaping (_ weather: Weather?,
                                          _ error: String?) -> Void) {

        var request = URLRequest(url: URL(string: url)!)
        request.httpMethod = "GET"

        requestWithRetry(with: request) {
                      (data, response, error, retriesLeft) in

            if let error = error {
                completion(nil, "\(error.localizedDescription) with \(retriesLeft) retries left")
                return
            }

            let statusCode = (response as! HTTPURLResponse).statusCode

            if statusCode == 200, let data = data {
                let weather = try? JSONDecoder().decode(
                                      Weather.self, from: data)
                completion(weather, nil)
            } else {
                completion(nil, "Error encountered: \(statusCode) with \(retriesLeft) retries left")
            }
        }
    }

    // **** This function is recursive, and will automatically retry
    private func requestWithRetry(with request: URLRequest,
                                  retries: Int = 3,
                                  completionHandler: @escaping
                                        (Data?, URLResponse?, Error?,
                                         _ retriesLeft: Int) -> Void) {

        let task = session.dataTask(with: request) {
                                   (data, response, error) in
            if error != nil {
                completionHandler(data, response, error, retries)
                return
            }

            let statusCode = (response as! HTTPURLResponse).statusCode

            if (200...299).contains(statusCode) {
                completionHandler(data, response, error, retries)
            } else if retries > 0 {
                print("Received status code \(statusCode) with \(retries) retries remaining. RETRYING VIA RECURSIVE CALL.")
                self.requestWithRetry(with: request,
                              retries: retries - 1,
                              completionHandler: completionHandler)
            } else {
                print("Received status code \(statusCode) with \(retries) retries remaining. EXIT WITH FAILURE.")
                completionHandler(data, response, error, retries)
            }
        }
        task.resume()
    }
}
```

The following illustrates what happens within the NetworkService as the repeated retries fail.

```markdown
Received status code 404 with 3 retries remaining. RETRYING VIA RECURSIVE CALL.

Received status code 404 with 2 retries remaining. RETRYING VIA RECURSIVE CALL.

Received status code 404 with 1 retries remaining. RETRYING VIA RECURSIVE CALL.

Received status code 404 with 0 retries remaining. EXIT WITH FAILURE.
```
