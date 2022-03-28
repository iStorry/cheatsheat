```swift
typealias DataSource = UICollectionViewDiffableDataSource<Section, Room>
private lazy var dataSource = makeDataSource()

collectionView.register(R.nib.channelCollectionCell)
collectionView.dataSource = dataSource
        

func makeDataSource() -> DataSource {
    let dataSource = DataSource(
        collectionView: collectionView,
        cellProvider: { (collectionView, indexPath, room) -> UICollectionViewCell? in
            let cell = collectionView.dequeueReusableCell(
                withReuseIdentifier: R.reuseIdentifier.channelCollectionCell,
                for: indexPath
            )!
            return cell
        }
    )
    return dataSource
}
```

### Network

```swift
var snapshot = NSDiffableDataSourceSnapshot<Section, Room>()
snapshot.appendSections([.main])
snapshot.appendItems(room.rooms, toSection: .main)
self?.dataSource.apply(snapshot, animatingDifferences: false)
```
