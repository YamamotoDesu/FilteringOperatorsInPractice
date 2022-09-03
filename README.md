# FilteringOperatorsInPractice

## RxSwift: Reactive Programming with Swift | raywenderlich.com
![image](https://user-images.githubusercontent.com/47273077/185172130-b3557025-c636-4a1b-8490-c900c8312b77.png)

### 1. Filtering elements you don't need
<img width="516" src="https://user-images.githubusercontent.com/47273077/187699767-e81d424e-b3de-481c-ad48-348d14b3cc9f.gif">

```swift
 @IBAction func actionAdd() {
    // images.accept(images.value + [UIImage(named: "IMG_1907.jpg")!])

    let photosViewController = storyboard!.instantiateViewController(
      withIdentifier: "PhotosViewController") as! PhotosViewController

    navigationController!.pushViewController(photosViewController, animated: true)

    let newPhotos = photosViewController.selectedPhotos.share()

    newPhotos
      .filter { newImage in
        return newImage.size.width > newImage.size.height
      }
      .filter { [weak self] newImage in
        let len = newImage.pngData()?.count ?? 0
        guard self?.imageCache.contains(len) == false else {
          return false
        }
        self?.imageCache.append(len)
        return true
      }
      .subscribe(
        onNext: { [weak self] newImage in
          guard let images = self?.images else { return }
          images.accept(images.value + [newImage])
        },
        onDisposed: {
          print("completed photo selection")
        }
      )
      .disposed(by: bag)
    
    newPhotos
      .ignoreElements()
      .subscribe(onCompleted: { [weak self] in
        self?.updateNavigationIcon()
      })
      .disposed(by: bag)
  }
```
  
## Keep taking elements while a condition is met
<img width="516" src="https://user-images.githubusercontent.com/47273077/188252826-b35f383d-7a83-47ae-b1c2-f21b2f7b1e60.gif">

```swift
   newPhotos
    .takeWhile { [weak self] image in
     let count = self?.images.value.count ?? 0
      return count < 3
    }
```

## PHPhotoLibarary authorization obervable
### Issue
<img width="516" src="https://user-images.githubusercontent.com/47273077/188253035-6d9dc3d5-cdcc-40b9-afe8-7ca14bda1814.gif">

```swift
extension PHPhotoLibrary {
  static var authorized: Observable<Bool> {
    return Observable.create { observer in
      DispatchQueue.main.async {
        if authorizationStatus() == .authorized {
          observer.onNext(true)
          observer.onCompleted()
        } else {
          observer.onNext(false)
          requestAuthorization { newStatus in
            observer.onNext(newStatus == .authorized)
            observer.onCompleted()
          }
        }
      }
      return Disposables.create()
    }
  }
}
```

```swift
  // MARK: View Controller
  override func viewDidLoad() {
    super.viewDidLoad()
    
    let autholized = PHPhotoLibrary.authorized.share()
    autholized
      .skipWhile { !$0 }
      .take(1)
      .subscribe(onNext: { [weak self] _ in
        self?.photos = PhotosViewController.loadPhotos()
        DispatchQueue.main.async {
          self?.collectionView?.reloadData()
        }
      })
      .disposed(by: bag)

  }
```
  
<img width="516" src="https://user-images.githubusercontent.com/47273077/188253936-9e204ec1-2f10-4b94-bd64-047e989e24b3.gif">

## Display an error message if the user does't grant access
<img width="516" src="https://user-images.githubusercontent.com/47273077/188254019-31ef8362-6d5d-424d-92c4-f3949d0cc04c.gif">

```swift
    autholized
//      .skip(1) // you can always skip the first element from the sequence, since it is the never the final one. ↓
      .distinctUntilChanged()
      .takeLast(1)
      .filter { !$0 }
      .subscribe(onNext: { [weak self] _ in
        guard let errorMessage = self?.errorMessage else { return }
        DispatchQueue.main.async(execute: errorMessage)
      })
      .disposed(by: bag)
 
   private func errorMessage() {
    alert(title: "No Access to the Camera Roll", text: "You can grant acess to Combinestagram from the Setting app")
      .subscribe(onCompleted: { [weak self] in
        self?.dismiss(animated: true, completion: nil)
        self?.navigationController?.popViewController(animated: true)
      })
      .disposed(by: bag)
  }
 ```
   
   
 ```swift
 extension UIViewController {
  func alert(title: String, text: String?) -> Completable {
    return Completable.create { [weak self] completable in
      let alertVC = UIAlertController(title: title, message: text, preferredStyle: .alert)
      alertVC.addAction(UIAlertAction(title: "Close", style: .default, handler: {_ in
        completable(.completed)
      }))
      self?.present(alertVC, animated: true, completion: nil)
      return Disposables.create {
        self?.dismiss(animated: true, completion: nil)
      }
    }
  }
}
```
<img width="516" src="https://user-images.githubusercontent.com/47273077/188254831-5ea3ba72-504e-42bf-963c-fea5e1163a40.gif">

