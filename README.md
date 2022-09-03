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
