# iOS-MVP (Inprogres)
iOS Mvp (Alamofire 5.0) Swift
@Mvp @iOS Development Architecture
@Mobile Architecture

# Requirements

* Swift 5
* Xcode 11+
* Alamofire 5.0.0 rc3 (https://github.com/Alamofire/Alamofire)

# Base Mvp Layer 

* BaseScreen

```Swift 
import UIKit

class BaseScreen<P>: UIViewController{
    
    var presenter: P?
    
    let baseData = BaseData.shared

    override func viewDidLoad() {
        super.viewDidLoad()
        initPresenter()
        initData()
        initUI()
    }
    
    func initData(){
        fatalError("Subclasses must implement initData()")

    }
    
    func initUI() {
        fatalError("Subclasses must implement initUI()")
    }
    
    func initPresenter() {
        fatalError("Subclasses must implement initPresenter()")
    }
    
    func getBaseData()-> BaseData {
        return baseData
    }
}



```

* BasePresenter
```Swift  
public class BasePresenter<V>{
    var baseView : V?
    
    let restClient = RestClient.sharedInstance
    
    init(baseView: V) {
        self.baseView = baseView
    }
   
    func detachView() {
           baseView = nil
    }
    
}
```
* BaseView
```Swift  
public protocol BaseView{
    func setUIData();
}

```

 

# Mvp Layer Example Using (Using Base Mvvm Layer)

* EventView
```Swift  
public protocol EventView: BaseView{
    func onSuccessVideo();
}
```

* EventPresenter

```Swift  
public class EventPresenter: BasePresenter<EventView> {
  
    func denemeRequest()  {
        self.baseView!.onSuccessVideo()
    }
}
```

* EventScreen

```Swift  
import UIKit

class EventScreen:BaseScreen<EventPresenter>,EventView{
  
    let baseData2:BaseData = BaseData.shared
    override func initPresenter() {
        presenter =  EventPresenter(baseView: self)
    }
    
    func onSuccessVideo() {
          
    }
      
    func setUIData() {
          
    }
      
}
```
    
# Network Layer

* BaseApiRequest
```Swift  
public protocol BaseApiRequest {
    var requestMethod: RequestHttpMethod?{ get }
    var requestPath: String {get}
    func request() -> URLRequest
}
```
* BaseApiRequest
    * Extension
    * NOT: swift protocol default parameter value (Protocol BaseApiRequest => extension BaseApiRequest)
```Swift  
extension BaseApiRequest{
    var enviroment: Environment {
        return Environment.Dev
    }
    public func request() -> URLRequest {
        let url: URL! = URL(string: baseUrl+requestPath)
        var request = URLRequest(url: url)
        switch requestMethod {
        case .Get:
            request.httpMethod = "GET"
            break
        case .Post:
            request.httpMethod = "POST"
            break
        default:
            request.httpMethod = "GET"
            break
        }
        return request
    }
    
    var baseUrl: String {
        switch enviroment {
        case .Prod:
            return "https://api.myjson.com/bins"
        case .Dev:
            return "https://api.myjson.com/bins"
        default:
            return "https://api.myjson.com/"
        }
    }
}
```
* BaseApiRequest
     * Enums
```Swift  
public enum RequestHttpMethod{
    case Get
    case Post
}

public enum Environment{
    case Prod
    case Dev
    case UAT
}
```

* GetEventListApiRequest
```Swift   
public class GetEventListApiRequest: BaseApiRequest {    
    public var requestMethod: RequestHttpMethod?
    public var requestPath: String = "/sltf6"
}
```
* GetEventResponse
```Swift   
public class GetEventResponse:Codable{
    
    
    var data : [Event]?
    
    enum CodingKeys: String, CodingKey {
          case data = "Data"
    }
 }
``` 


 Alamofire Generic Request 
* RestClient

```Swift   
private func sendRequest<T:Codable>(_ request:BaseApiRequest,_ type :T.Type,successHandler:@escaping(T)->(),failHandler:@escaping(Error)->()){
        AF.request(request.request()).responseDecodable { (response:AFDataResponse<T>) in
             switch response.result{
                       case .success(let responseEventList):
                           successHandler(responseEventList)
                           print("success")
                       case .failure(let error):
                           failHandler(error)
                           print("fail")
            }
        }
    }
 ```
 
* IServiceHandler
    * Protocol

```Swift   
public protocol IServiceHandler {
    func getEventList(successHandler:@escaping(GetEventResponse)->(),
                      failHandler:@escaping(Error)->())

}
 ```
