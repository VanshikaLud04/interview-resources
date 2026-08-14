# Low-Level Design (OOD) Questions — By Company

## Amazon / Wayfair
- **Multi-Floor Parking Lot**
  - **Key Patterns**: Singleton (Parking Manager), Factory (Ticket/Vehicle).
  - **Core Classes**: `ParkingLot`, `Level`, `ParkingSpot`, `Vehicle`, `Ticket`.
  - **Considerations**: Concurrency when assigning spots, pricing strategies.
- **Database Partition Split-and-Merge Manager**
  - **Key Patterns**: State (Partition Status), Command (Split/Merge Operations).
  - **Core Classes**: `PartitionManager`, `Partition`, `SplitAction`, `MergeAction`.
  - **Considerations**: Transactionality, rollback on failure, handling active reads/writes.
- **Streaming Log Handler**
  - **Key Patterns**: Observer (Subscribers), Decorator (Filters).
  - **Core Classes**: `LogStream`, `Filter`, `Subscriber`, `LogProcessor`.
  - **Considerations**: High throughput, backpressure, thread-safe buffering.
- **Playlist Source Mixer**
  - **Key Patterns**: Strategy (Mixing Algorithms), Iterator.
  - **Core Classes**: `PlaylistMixer`, `Source`, `Track`, `MixingStrategy`.
  - **Considerations**: Handling sources of different lengths, randomness vs fairness.
- **File-System Search with Symlink Safety**
  - **Key Patterns**: Composite (Files/Directories), Visitor (Search Algorithm).
  - **Core Classes**: `FileSystemNode`, `File`, `Directory`, `SearchVisitor`.
  - **Considerations**: Cycle detection for symlinks, filtering by metadata.
- **Music Search and Playback**
  - **Key Patterns**: Facade (System Entry Point), Builder (Search Queries).
  - **Core Classes**: `MusicPlayer`, `Library`, `Song`, `SearchCriteria`.
  - **Considerations**: In-memory indexing vs DB query, caching frequent searches.
- **Extensible Multi-Channel Alerting Platform**
  - **Key Patterns**: Strategy (Notification Method), Observer.
  - **Core Classes**: `AlertManager`, `Notifier (Interface)`, `EmailNotifier`, `SMSNotifier`.
  - **Considerations**: Rate limiting, retry mechanisms, failure fallbacks.

## Goldman Sachs
- **In-Memory Job Scheduler**
  - **Key Patterns**: Command, Observer (Job Completion).
  - **Core Classes**: `Scheduler`, `Job`, `WorkerThread`, `CronParser`.
  - **Considerations**: Thread pool management, recurring vs one-time jobs, handling job failures.

## Shopify
- **Extensible Text Editor**
  - **Key Patterns**: Memento (Undo/Redo), Command (Edit Actions).
  - **Core Classes**: `Document`, `TextEditor`, `Command`, `History`.
  - **Considerations**: Memory efficiency for history, concurrent edits.

## Uber
- **Parking Lot Design and Code**
  - *(See Amazon Multi-Floor Parking Lot)*
- **Ride Dispatch and Trip Lifecycle**
  - **Key Patterns**: State (Trip Status), Observer (Driver/Rider updates).
  - **Core Classes**: `TripManager`, `Trip`, `Rider`, `Driver`, `MatchingStrategy`.
  - **Considerations**: Geolocation indexing (QuadTree/S2), concurrent dispatch requests.

## Bloomberg
- **Extensible Cache Eviction Policies**
  - **Key Patterns**: Strategy (Eviction Algorithm), Factory.
  - **Core Classes**: `Cache`, `EvictionPolicy`, `LRUPolicy`, `LFUPolicy`.
  - **Considerations**: Time complexity of operations (aim for $O(1)$), thread safety.

## Asana
- **Rectangular Jigsaw Puzzle**
  - **Key Patterns**: Builder, Strategy (Matching).
  - **Core Classes**: `Puzzle`, `Piece`, `Edge`, `Solver`.
  - **Considerations**: Efficient neighbor matching, rotating pieces.

## Databricks
- **Locking Key-Value Store with Batch Writes**
  - **Key Patterns**: Command (Batch Actions), Singleton.
  - **Core Classes**: `KVStore`, `Transaction`, `LockManager`.
  - **Considerations**: Deadlock prevention, isolation levels, atomicity.
- **WAL-Backed Batch Log Writer**
  - **Key Patterns**: Decorator, Strategy (Flush Policy).
  - **Core Classes**: `WALWriter`, `LogEntry`, `Buffer`, `Storage`.
  - **Considerations**: Crash recovery, asynchronous flushing.

## Susquehanna
- **Cash Register Inventory and Profit**
  - **Key Patterns**: Observer, Strategy (Pricing/Discounts).
  - **Core Classes**: `Register`, `Inventory`, `Item`, `Transaction`.
  - **Considerations**: Precision for currency (use Decimal/BigDecimal), concurrent transactions.

## Pinterest
- **Blackjack Table Game**
  - **Key Patterns**: State (Game/Player State), Factory (Deck).
  - **Core Classes**: `Game`, `Player`, `Dealer`, `Deck`, `Card`.
  - **Considerations**: Rule edge cases (Split, Double Down), deck shuffling.

## Meta
- **In-Memory Tally Service**
  - **Key Patterns**: Singleton, Decorator (Metrics).
  - **Core Classes**: `TallyManager`, `Counter`, `RateLimiter`.
  - **Considerations**: Extremely high concurrency, atomic operations, bucketing for time-series.

## Tekion
- **Configurable Logging Framework**
  - **Key Patterns**: Chain of Responsibility, Strategy (Formatting/Appenders).
  - **Core Classes**: `Logger`, `Appender`, `Formatter`, `LogEvent`.
  - **Considerations**: Asynchronous logging, log rotation, dynamic log level changes.

## Adobe
- **Mutable Infinite Scroll**
  - **Key Patterns**: Iterator, Observer.
  - **Core Classes**: `DataStore`, `ScrollController`, `ViewPort`.
  - **Considerations**: Prefetching, memory management (evicting off-screen data).
- **Network-Backed Search Suggestions**
  - **Key Patterns**: Proxy (Cache), Strategy (Debouncing).
  - **Core Classes**: `SuggestionService`, `NetworkClient`, `Cache`.
  - **Considerations**: Debouncing, handling out-of-order network responses, Trie for local cache.
