#### E có vài chỗ thắc mắc
1. Tại sao lại để default cho viewModel và UseCase a nhỉ?
Theo e thấy thì mình có dùng Inject Dependency rồi thì bỏ cái này vô chi a? 
Với lại cái này là chưa có thêm param truyền vào, nếu truyền vào nhiều thì e thấy không ok chỗ này lắm.
Nếu inject thiếu mình cũng không biết.
``` swift
var viewModel: FirstViewModel! = FirstViewModel()
```
``` swift
class FirstViewModel {
    private let authUseCase: AuthenUseCaseable!

    var disposeBag = DisposeBag()
    
    typealias Info = (username: String, password: String)
    
    init(useCase: AuthenUseCaseable = AuthenUsecaseImplement()) {
        self.authUseCase = useCase
    }
}
```

2. Những cái như chỉ có emit một lần, 1 là thành công 2 là thất bại, không có emit nhiều lần thì e nghĩ mình nên sử dụng `Single`
``` swift
func login(phone: String, password: String) -> Observable<User?>
=> func login(phone: String, password: String) -> Single<User?>
```
3. Bên usecase e nghĩ nên có repository, từ đây repository là trung gian adapter giữa use case và entities 
![image](https://user-images.githubusercontent.com/34368022/92202714-c6c67500-eea9-11ea-99f7-351f5f8bffed.png)
```
Ở repository sẽ có protocol Local Datasource và Remove Datasource,
Local Datasource -> Chịu trách nhiệm dữ liệu ở local, như lưu dữ liệu vào persistance
Remote Datasource -> Làm việc với API. 
Ta có thể implement và thay thế dễ dàng local DataSource bằng nhiều cách implement khác nhau như: Core Data, Disk, hoặc lưu tạm để Fake.
Tương tự với RemoteDataSource bằng nhiều cách implement khác nhau như: Call API, thay thế bằng GraphQL, hay là fake để API...
Thì cần sửa hay thay đổi ta chỉ cần thay đổi implement không phải đụng tới những cái trước đó. 
```
Ở dưới là ví dụ của e, a xem qua thử, e chưa hiểu, hay hiểu sai chỗ nào a chỉ dùm e với. :bow:
``` swift
// Data Source
protocol ProfilesLocalDataSource {
    func save(user: User?)
    func remove()
}

protocol ProfilesRemoteDataSource {
    func login(phone: String, password: String) -> Single<User?>
}

// Repository
protocol ProfilesRepository {
    func login(phone: String, password: String) -> Single<User?>
}

class ProfilesRepositoryImpl: ProfilesRepository {
    
    let localDataSouce: ProfilesLocalDataSource
    //      => save to core data
    //      => save to disk
    //      => Fake save to temp
    let remoteDataSouce: ProfilesRemoteDataSource
    //      => Call API by Alamofire
    //      => Use GraphQL
    //      => Fake call API
    
    init(localDataSouce: ProfilesLocalDataSource, remoteDataSouce: ProfilesRemoteDataSource) {
        self.localDataSouce = localDataSouce
        self.remoteDataSouce = remoteDataSouce
    }
    
    func login(phone: String, password: String) -> Single<User?> {
        // Login success => save to local
        return remoteDataSouce.login(phone: phone, password: password)
            .do(onSuccess: localDataSouce.save(user:))
    }
}


// UseCase
protocol AuthenUseCaseable: class {
    func login(phone: String, password: String) -> Single<User?>
}

class AuthenUsecaseImplement: AuthenUseCaseable {
    let profilesRepository: ProfilesRepository
    
    init(profilesRepository: ProfilesRepository) {
        self.profilesRepository = profilesRepository
    }
    
    func login(phone: String, password: String) -> Single<User?> {
        return profilesRepository.login(phone: phone, password: password)
    }
}
```
