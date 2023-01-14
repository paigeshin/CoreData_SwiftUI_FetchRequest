# CoreData_SwiftUI_FetchRequest

### App

```swift
import SwiftUI

@main
struct SwiftUI_NSFetchRequestApp: App {
    
    private var coreDataManager = CoreDataManager.shared
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(\.managedObjectContext, self.coreDataManager.viewContext)
        }
    }
}

```

### CoreDataManager

```swift

import Foundation
import CoreData

class CoreDataManager {
    
    let persistentContainer: NSPersistentContainer
    static var shared: CoreDataManager = CoreDataManager()
    
    var viewContext: NSManagedObjectContext {
        return self.persistentContainer.viewContext
    }
    
    private init() {
        self.persistentContainer = NSPersistentContainer(name: "MovieAppModel")
        self.persistentContainer.loadPersistentStores { desc, error in
            if let error {
                fatalError("Unable to initialize Core Data stores \(error)")
            }
        }
    }
    
}

```

### Movie Extension
```swift
import Foundation
import CoreData

extension Movie {
    
    static var all: NSFetchRequest<Movie> = {
        let request: NSFetchRequest<Movie> = Movie.fetchRequest()
        request.sortDescriptors = [NSSortDescriptor(key: "title", ascending: true)]
        return request
    }()
    
    static var byRating: NSFetchRequest<Movie> = {
        let request: NSFetchRequest<Movie> = Movie.fetchRequest()
        request.sortDescriptors = [NSSortDescriptor(key: "rating", ascending: false)]
        return request
    }()
    
}

```

### ContentView

```swift
import SwiftUI

struct ContentView: View {
    
    @Environment(\.managedObjectContext) private var viewContext
    
//    @FetchRequest(entity: Movie.entity(),
//                  sortDescriptors: [NSSortDescriptor(key: "title", ascending: true)])
    @FetchRequest(fetchRequest: Movie.byRating)
    var movies: FetchedResults<Movie>
    @State private var movieName: String = ""
    
    var body: some View {
        VStack {
            
            HStack {
                TextField("Movie Name", text: self.$movieName)
                    .textFieldStyle(.roundedBorder)
                Button("Add Movie") {
                    let movie = Movie(context: self.viewContext)
                    movie.title = self.movieName
                    movie.rating = Int16.random(in: 1...5)
                    try? self.viewContext.save()
                    self.movieName = ""
                }
            } //: HSTACK
            
            
            List(self.movies, id: \.self) { movie in
                HStack {
                    Text(movie.title ?? "")
                    Spacer()
                    HStack {
                        Text("\(movie.rating)")
                        Image(systemName: "star.fill")
                            .foregroundColor(.yellow)
                    }
                } //: HSTACK
            } //: LIST
            .navigationTitle("Movies")
            
        }
        .padding()
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
            .environment(\.managedObjectContext, CoreDataManager.shared.viewContext)
    }
}
```
