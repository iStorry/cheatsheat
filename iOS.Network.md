# Network


### Endpoints.swift

```swift
struct Endpoint {
    var path: String
    var queryItems: [URLQueryItem] = []
}

extension Endpoint {
    var url: URL {
        var components = URLComponents()
        components.scheme = "http"
        components.host = "192.168.1.203"
        components.port = 3000
        components.path = "/api/v1" + path
        // for query
        # components.queryItems = queryItems
        
        guard let url = components.url else {
            preconditionFailure("Invalid URL components: \(components)")
        }
        
        return url
    }
    
    // Custom Headers
    var headers: [String: Any] {
        return [
            "app-id": "??"
        ]
    }
}


extension Endpoint {
    static var list: Self {
        return Endpoint(path: "/list")
    }
    
}

```

### NetworkControllerProtocol.swift

```swift
protocol NetworkControllerProtocol: AnyObject {
    typealias Headers = [String: Any]
    func get<T>(type: T.Type, url: URL, headers: Headers) -> AnyPublisher<T, Error> where T: Decodable
    func post<T, U>(url: URL, headers: Headers, payload: T) -> AnyPublisher<U, Error> where T: Codable, U: Decodable
}

final class NetworkController: NetworkControllerProtocol {
    
    func get<T: Decodable>(type: T.Type, url: URL, headers: Headers) -> AnyPublisher<T, Error> {
        var urlRequest = URLRequest(url: url)
        headers.forEach { (key, value) in
            if let value = value as? String {
                urlRequest.setValue(value, forHTTPHeaderField: key)
            }
        }
        return URLSession.shared.dataTaskPublisher(for: urlRequest).map(\.data).decode(type: T.self, decoder: JSONDecoder()).eraseToAnyPublisher()
    }
    
    func post<T, U>(url: URL, headers: Headers, payload: T)  -> AnyPublisher<U, Error> where T: Codable, U: Decodable {
        var urlRequest = URLRequest(url: url)
        headers.forEach { (key, value) in
            if let value = value as? String {
                urlRequest.setValue(value, forHTTPHeaderField: key)
            }
        }
               
        let encoder = JSONEncoder()
        do {
            let jsonData = try encoder.encode(payload)
            urlRequest.httpBody = jsonData
        } catch {
            print("\(error.localizedDescription)")
        }
        return URLSession.shared.dataTaskPublisher(for: urlRequest).map(\.data).decode(type: U.self, decoder: JSONDecoder()).eraseToAnyPublisher()
    }
}
```

### RoomControllerProtocol.swift

```swift
protocol RoomControllerProtocol: AnyObject {
    var networkController: NetworkControllerProtocol { get }
    func getRoomsList() -> AnyPublisher<RoomsResponse, Error>
}

final class RoomLogicController: RoomControllerProtocol {
    
    let networkController: NetworkControllerProtocol
    
    init(networkController: NetworkControllerProtocol) {
        self.networkController = networkController
    }
    
    func getRoomsList() -> AnyPublisher<RoomsResponse, Error> {
        let endpoint = Endpoint.roomsList
        return networkController.get(type: RoomsResponse.self, url: endpoint.url, headers: endpoint.headers)
    }
}
```

### Usage

```swift
private let networkController = NetworkController()
private var subscriptions = Set<AnyCancellable>()
private func networkRequest() {
    let roomLogicController = RoomLogicController(
        networkController: networkController
    )
    roomLogicController.getRoomsList().sink(receiveCompletion: { completion in
        switch completion {
        case .failure(let error):
            print("Error : \(error.localizedDescription)")
        case .finished:
            print("Finished")
        }
    }) { [weak self] room in
        var snapshot = NSDiffableDataSourceSnapshot<Section, Room>()
        snapshot.appendSections([.main])
        snapshot.appendItems(room.rooms, toSection: .main)
        self?.dataSource.apply(snapshot, animatingDifferences: false)
    }.store(in: &subscriptions)
}
```
