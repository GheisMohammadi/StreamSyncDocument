# Harmony Stream Sync Guide for Validators

## Overview

Harmony is transitioning from DNS-based synchronization to a new, decentralized **Stream Sync** system. This guide explains what Stream Sync is, why it's better than the current DNS sync, and how it benefits the Harmony network.

## What is Stream Sync?

Stream Sync is a peer-to-peer (P2P) synchronization protocol that allows Harmony nodes to sync with each other directly through neighbor connections, rather than relying on centralized DNS servers. It's built on top of the existing P2P network infrastructure and uses a staged approach to efficiently download blockchain data.

**Current Implementation Status**: Stream Sync is currently implemented with Full Sync mode only. Fast Sync and Snap Sync modes are reserved for future development and are not available for use.

**Current Deployment Status**: Stream Sync is fully operational on devnet and testnet. On mainnet, Stream Sync server is enabled by default, while the client (full stream sync functionality) is optional for validators.

## Current DNS Sync vs. Stream Sync

### DNS Sync (Current System)
DNS Sync is a centralized synchronization method where:
- All nodes connect to a few static DNS servers managed by Harmony
- These servers act as central points for block synchronization
- All network load is concentrated on these few DNS nodes
- If DNS servers go down, the entire network's sync capability is affected

**Problems with DNS Sync:**
- **Single Point of Failure**: If DNS servers are unavailable, new nodes cannot sync
- **Centralized Control**: Harmony manages all sync infrastructure
- **Scalability Issues**: As the network grows, DNS servers become bottlenecks
- **Network Congestion**: All sync traffic goes through limited endpoints
- **Geographic Limitations**: Nodes far from DNS servers experience higher latency

### Stream Sync (New System)
Stream Sync is a decentralized approach where:
- Nodes sync directly with their P2P neighbors
- No central servers are required
- Network load is distributed across all participating nodes
- Each node can contribute to the sync process

## How Stream Sync Works

### 1. Peer Discovery
- Nodes discover each other through the existing P2P network
- Each node maintains connections with multiple neighbors
- No need for centralized DNS resolution

### 2. Staged Synchronization
Stream Sync uses a staged approach for efficient data transfer. Currently, only Full Sync is fully implemented and available for use.

**Full Sync Stages:**
1. **Heads**: Retrieve chain heads and latest block information
2. **SyncEpoch**: Synchronize only the last block of each epoch
3. **ShortRange**: Perform short-range synchronization for recent blocks
4. **BlockHashes**: Download block hashes for verification
5. **BlockBodies**: Download actual block data and verify transaction hashes
6. **States**: Update blockchain state from downloaded blocks
7. **Finish**: Finalize all changes and complete the sync cycle

**Note**: Fast Sync and Snap Sync modes are reserved for future implementation and are not currently available.

Each stage must complete successfully before moving to the next, ensuring data integrity and consistency.

### 3. Concurrent Downloads
- Multiple streams can download data simultaneously
- Configurable concurrency levels for optimal performance
- Automatic load balancing across available peers

### 4. Fault Tolerance
- If one peer becomes unavailable, others take over
- Automatic retry mechanisms for failed downloads
- Stream health monitoring and replacement
- Cooldown periods for failed streams to prevent rapid reconnection attempts
- Reserved stream pool for instant replacement of failed connections

## Benefits of Stream Sync

### 1. **Decentralization**
- No single point of failure
- Network continues to function even if some nodes go offline
- True P2P architecture aligns with blockchain principles

### 2. **Scalability**
- Network capacity grows with the number of nodes
- No central bottlenecks limiting sync performance
- Better handling of network growth

### 3. **Reliability**
- Multiple redundant sync paths
- Automatic failover between peers
- More resilient to network issues

### 4. **Performance**
- Lower latency for most nodes (closer to neighbors)
- Better bandwidth utilization
- Reduced network congestion

### 5. **Cost Efficiency**
- No need to maintain expensive centralized infrastructure
- Distributed resource usage across the network
- Lower operational costs for Harmony

### 6. **Geographic Distribution**
- Nodes can sync with geographically closer peers
- Reduced cross-continental traffic
- Better performance for global networks

### 7. **Security**
- No central sync servers that could be targeted
- Distributed trust model
- Better resistance to DDoS attacks

## Technical Implementation

### Stream Management
- **Stream Manager**: Handles multiple concurrent sync streams
- **Protocol Matching**: Ensures compatibility between nodes
- **Rate Limiting**: Prevents abuse and ensures fair resource usage
- **Reserved Stream Pool**: Maintains backup streams for instant replacement
- **Cooldown Management**: Implements progressive backoff for failed connections

### Configuration Options
Stream Sync provides extensive configuration options to optimize performance for different network conditions and node types.

#### Core Sync Configuration
- **Enabled** (default: `true`): Enable the stream sync protocol
- **SyncMode** (default: `0`): Synchronization mode (0=FullSync, 1=FastSync, 2=SnapSync - FastSync and SnapSync are reserved for future implementation)
- **Client** (default: `false`): Start the sync downloader client
- **Concurrency** (default: `6` for mainnet, `3-4` for other networks): Number of concurrent sync requests
- **MinPeers** (default: `6` for mainnet, `3-4` for other networks): Minimum streams required to start a sync task
- **InitStreams** (default: `8` for mainnet, `3-4` for other networks): Minimum streams required for initial bootstrap
- **MaxAdvertiseWaitTime** (default: `60` minutes for mainnet, `2-5` minutes for other networks): Maximum time between protocol advertisements

#### Stream Manager Configuration
- **DiscSoftLowCap** (default: `8` for mainnet, `3-4` for other networks): When stream count drops below this value, discovery is triggered during routine checks
- **DiscHardLowCap** (default: `6` for mainnet, `3-4` for other networks): When removing streams, if count drops below this value, discovery is triggered immediately
- **DiscHighCap** (default: `128` for mainnet, `1024` for other networks): Upper limit of streams in one sync protocol
- **DiscBatch** (default: `8` for mainnet, `3-8` for other networks): Size of each discovery batch

#### Staged Stream Sync Configuration
- **DoubleCheckBlockHashes** (default: `false`): Enable double-checking of block hashes for additional verification
- **MaxBlocksPerSyncCycle** (default: `512`): Maximum blocks to sync in each cycle (0 means all blocks in one full cycle)
- **MaxBackgroundBlocks** (default: `512`): Maximum blocks to download in background process in turbo mode
- **InsertChainBatchSize** (default: `128`): Number of blocks to build a batch and insert to chain in staged sync
- **VerifyAllSig** (default: `false`): Whether to verify signatures for all blocks
- **VerifyHeaderBatchSize** (default: `100`): Batch size to verify block headers before inserting to chain
- **MaxMemSyncCycleSize** (default: `1024`): Maximum number of blocks to use a single transaction for staged sync
- **UseMemDB** (default: `true`): Use memory database by default (set to false to use disk)
- **LogProgress** (default: `false`): Log the full sync progress in console
- **DebugMode** (default: `false`): Log every single process and error for debugging (development use only)

#### Network-Specific Defaults
**Mainnet:**
- Concurrency: 6, MinPeers: 6, InitStreams: 8
- DiscSoftLowCap: 8, DiscHardLowCap: 6, DiscHighCap: 128, DiscBatch: 8
- MaxAdvertiseWaitTime: 60 minutes

**Testnet:**
- Concurrency: 3, MinPeers: 3, InitStreams: 3
- DiscSoftLowCap: 3, DiscHardLowCap: 3, DiscHighCap: 1024, DiscBatch: 3
- MaxAdvertiseWaitTime: 5 minutes

**Localnet:**
- Concurrency: 4, MinPeers: 4, InitStreams: 4
- DiscSoftLowCap: 4, DiscHardLowCap: 4, DiscHighCap: 1024, DiscBatch: 8
- MaxAdvertiseWaitTime: 5 minutes

**Partner Networks:**
- Concurrency: 3, MinPeers: 3, InitStreams: 3
- DiscSoftLowCap: 3, DiscHardLowCap: 3, DiscHighCap: 1024, DiscBatch: 5
- MaxAdvertiseWaitTime: 5 minutes

#### Advanced Stream Management
- **MaxReservedStreams** (default: `100`): Maximum number of reserved streams that can be maintained
- **RemovalCooldownDuration** (default: `5` minutes): Base cooldown period for failed streams
- **MaxRemovalCooldownDuration** (default: `60` minutes): Maximum cooldown period for critical errors
- **SetupConcurrency** (default: `16`): Limits concurrent stream setup goroutines
- **CheckInterval** (default: `30` seconds): Interval for checking stream count and triggering discovery
- **DiscoveryTimeout** (default: `10` seconds): Timeout for one batch of discovery
- **ConnectTimeout** (default: `60` seconds): Timeout for setting up a stream with a discovered peer

#### Configuration File Example
These settings can be configured in your `harmony.conf` file under the `[Sync]` section:

```toml
[Sync]
  Enabled = true
  SyncMode = 0
  Client = false
  Concurrency = 6
  MinPeers = 6
  InitStreams = 8
  MaxAdvertiseWaitTime = 60
  DiscSoftLowCap = 8
  DiscHardLowCap = 6
  DiscHighCap = 128
  DiscBatch = 8

  [Sync.StagedSyncCfg]
    DoubleCheckBlockHashes = false
    MaxBlocksPerSyncCycle = 512
    MaxBackgroundBlocks = 512
    InsertChainBatchSize = 128
    VerifyAllSig = false
    VerifyHeaderBatchSize = 100
    MaxMemSyncCycleSize = 1024
    UseMemDB = true
    LogProgress = false
    DebugMode = false
```

#### Command Line Flags
You can also override these settings using command line flags:

```bash
# Enable stream sync
--sync

# Set sync mode (0=FullSync, 1=FastSync, 2=SnapSync - FastSync and SnapSync are reserved)
--sync.mode 0

# Set concurrency
--sync.concurrency 8

# Set minimum peers
--sync.min-peers 8

# Set initial streams
--sync.init-peers 10

# Set discovery batch size
--sync.disc.batch 16
```

## Peer Identification and Bad Actor Detection

### How Stream Sync Identifies Correct Peers
Stream Sync uses multiple mechanisms to identify and connect to reliable peers:

1. **Protocol Compatibility**: Only peers with matching protocol versions, service types, network types, and shard IDs can establish streams
2. **Trusted Peer List**: Pre-configured trusted peers are prioritized for connections
3. **Discovery Service**: Uses DHT (Distributed Hash Table) to find peers providing the sync service
4. **Stream Validation**: Each incoming stream undergoes sanity checks before acceptance

### What Causes a Stream to be Considered a Bad Actor
Streams can be marked as problematic and removed for several reasons:

1. **Critical Errors**: Protocol violations, invalid data, or security issues
2. **Connection Failures**: Repeated connection drops or network timeouts
3. **Performance Issues**: Slow response times or excessive resource usage
4. **Invalid Responses**: Sending incorrect or malicious data
5. **Rate Limit Violations**: Exceeding allowed request limits
6. **Protocol Mismatches**: Incompatible protocol versions or configurations

### Cooldown Process and Failures
When a stream fails, Stream Sync implements a sophisticated cooldown system:

1. **Progressive Backoff**: Each failure increases the cooldown duration
2. **Failure Counting**: Tracks the number of failures per stream
3. **Cooldown Periods**: 
   - First failure: No cooldown (to avoid penalizing one-off disconnects)
   - Subsequent failures: Increasing cooldown duration (5 minutes × failure count)
   - Maximum cooldown: 60 minutes for critical errors
4. **Automatic Recovery**: Streams can reconnect after cooldown expires
5. **Trusted Peer Protection**: Trusted peers are not removed even for critical errors

### Reserved List and Instant Reconnection
Stream Sync maintains a reserved stream pool for instant replacement:

1. **Reserved Stream Pool**: Up to 100 backup streams can be maintained
2. **Automatic Promotion**: When main streams fail, reserved streams are immediately promoted
3. **Instant Reconnection**: No waiting period for reserved streams to become active
4. **Load Balancing**: Reserved streams help maintain optimal stream counts
5. **Discovery Triggering**: If stream count drops below thresholds, discovery is automatically triggered

### Discovery and Challenges
The peer discovery system uses several strategies to find and connect to peers:

1. **DHT Discovery**: Uses libp2p's Kademlia DHT to find peers advertising the sync service
2. **Trusted Peer Connections**: Prioritizes connections to pre-configured trusted peers
3. **Batch Discovery**: Discovers peers in configurable batches to avoid overwhelming the network
4. **Discovery Challenges**:
   - **Network Partitioning**: DHT may not find all available peers
   - **Protocol Versioning**: Peers must have compatible protocol versions
   - **Geographic Distribution**: Peers may be geographically clustered
   - **Resource Constraints**: Limited bandwidth and connection capacity
5. **Discovery Cooldown**: Implements cooldown periods to prevent excessive discovery attempts
6. **Connection Limits**: Limits concurrent connection attempts to prevent resource exhaustion

## Migration Strategy

### Phase 1: Coexistence
- **Devnet and Testnet**: Stream Sync is fully enabled and operational
- **Mainnet**: Stream Sync server is enabled by default for all nodes
- **Mainnet Validators**: Can optionally enable Stream Sync client for full stream sync functionality
- **Fallback**: Nodes without Stream Sync client enabled continue using DNS sync
- **Gradual Transition**: Validators can test Stream Sync while maintaining DNS sync compatibility

### Phase 2: Primary Sync
- Stream Sync becomes the default method
- DNS sync remains as fallback
- Performance monitoring and optimization

### Phase 3: DNS Sync Deprecation
- Complete transition to Stream Sync
- DNS sync infrastructure can be decommissioned
- Full decentralization achieved

## Validator Impact

### What Changes for Validators
- **No Action Required**: Stream Sync is automatically enabled
- **Better Performance**: Faster synchronization with neighbors
- **Improved Reliability**: Less dependency on external services
- **Network Contribution**: Your node helps others sync

### What Stays the Same
- **Consensus Process**: No changes to block validation
- **Staking Operations**: All staking functionality remains unchanged
- **RPC Endpoints**: Same API interfaces for applications
- **Monitoring**: Existing monitoring tools continue to work

### Performance Expectations
- **Faster Sync**: Initial sync should be noticeably faster
- **Better Stability**: Fewer sync interruptions
- **Lower Latency**: Reduced sync delays during network issues

## Monitoring and Troubleshooting

### Health Indicators
- **Stream Count**: Number of active sync streams
- **Reserved Stream Count**: Number of backup streams available
- **Sync Progress**: Current synchronization status and stage
- **Peer Connections**: Number of connected neighbors
- **Download Speed**: Blocks per second being processed
- **Stream Failures**: Count of failed streams and failure reasons
- **Discovery Status**: Active discovery attempts and success rates
- **Cooldown Status**: Streams currently in cooldown periods

### Common Issues
- **Insufficient Peers**: Ensure adequate P2P connections and check discovery status
- **Stream Failures**: Automatic recovery with cooldown periods, but may indicate network issues
- **Slow Sync**: Check bandwidth, peer quality, and stream concurrency settings
- **Discovery Failures**: May indicate DHT issues or network partitioning
- **Cooldown Loops**: Repeated failures may indicate persistent network problems
- **Reserved Stream Exhaustion**: When backup streams are depleted, sync performance may degrade

### Logs and Debugging
- Stream Sync provides detailed logging for all stages and operations
- Performance metrics available through monitoring and Prometheus endpoints
- Debug mode for troubleshooting sync issues and stream management
- Stream health statistics including failure counts and failure distributions
- Discovery logs showing peer finding attempts and connection results
- Cooldown tracking for understanding stream removal patterns

## Metrics and Monitoring

Stream Sync provides comprehensive metrics and monitoring capabilities through Prometheus endpoints. These metrics help validators monitor the health and performance of their sync operations.

### Core Sync Metrics

#### Staged Stream Sync Metrics
- **`hmy_staged_stream_sync_consensus_trigger`**: Number of times consensus triggered download tasks
- **`hmy_staged_stream_sync_num_blocks_synced_long_range`**: Number of blocks synced in long-range sync
- **`hmy_staged_stream_sync_num_blocks_failed_long_range`**: Number of blocks that failed to insert into chain
- **`hmy_staged_stream_sync_num_short_range`**: Number of short-range sync operations triggered
- **`hmy_staged_stream_sync_num_epoch_sync`**: Number of epoch block sync operations triggered
- **`hmy_staged_stream_sync_failed_download`**: Number of failed download operations
- **`hmy_staged_stream_sync_num_blocks_inserted_short_range`**: Histogram of blocks inserted per short-range sync
- **`hmy_staged_stream_sync_num_blocks_inserted_epoch_sync`**: Histogram of blocks inserted per epoch sync
- **`hmy_staged_stream_sync_num_blocks_inserted_beacon_helper`**: Number of blocks inserted from beacon helper

#### Stream Protocol Metrics
- **`hmy_stream_sync_client_request`**: Number of outgoing requests as a client
- **`hmy_stream_sync_failed_client_request`**: Number of failed outgoing requests
- **`hmy_stream_sync_client_request_delay`**: Histogram of client request delays
- **`hmy_stream_sync_server_request`**: Number of incoming requests as a server

### Stream Management Metrics

#### Stream Counters and Gauges
- **`hmy_stream_num_streams`**: Current number of connected streams
- **`hmy_stream_num_reserved_streams`**: Current number of reserved streams
- **`hmy_stream_added_streams`**: Total number of streams added
- **`hmy_stream_removed_streams`**: Total number of streams removed
- **`hmy_stream_stream_critical_errors`**: Number of critical errors causing stream removal
- **`hmy_stream_stream_removal_reasons`**: Distribution of stream removal reasons

#### Discovery and Connection Metrics
- **`hmy_stream_discover`**: Number of peer discovery attempts
- **`hmy_stream_discover_peers`**: Number of peers successfully discovered and connected
- **`hmy_stream_setup_stream_duration`**: Histogram of stream setup durations

#### Stream Health Metrics
- **`hmy_stream_bytes_read`**: Total bytes read from streams
- **`hmy_stream_bytes_write`**: Total bytes written to streams
- **`hmy_stream_msg_read`**: Total messages read from streams
- **`hmy_stream_msg_write`**: Total messages written to streams
- **`hmy_stream_msg_read_failed`**: Number of failed message reads
- **`hmy_stream_msg_write_failed`**: Number of failed message writes

### Request Management Metrics

#### Request State Metrics
- **`hmy_stream_request_manager_num_connected_streams`**: Number of streams connected to request manager
- **`hmy_stream_request_manager_num_available_streams`**: Number of streams available for requests
- **`hmy_stream_num_waiting_requests`**: Number of requests waiting in queue
- **`hmy_stream_num_pending_requests`**: Number of requests currently being processed

#### Rate Limiting Metrics
- **`hmy_stream_num_server_request`**: Number of incoming requests as server
- **`hmy_stream_server_request_delay`**: Histogram of server request processing delays

### Monitoring Dashboard Queries

#### Stream Health Check
```promql
# Stream count by protocol
hmy_stream_num_streams{topic="harmony/sync/mainnet/0/1.0.0"}

# Reserved stream count
hmy_stream_num_reserved_streams{topic="harmony/sync/mainnet/0/1.0.0"}

# Stream failure rate
rate(hmy_stream_stream_critical_errors{topic="harmony/sync/mainnet/0/1.0.0"}[5m])
```

#### Sync Performance Monitoring
```promql
# Blocks synced per second
rate(hmy_staged_stream_sync_num_blocks_synced_long_range[5m])

# Failed downloads
rate(hmy_staged_stream_sync_failed_download[5m])

# Request latency
histogram_quantile(0.95, rate(hmy_stream_sync_client_request_delay_bucket[5m]))
```

#### Request Queue Health
```promql
# Request queue depth
hmy_stream_num_waiting_requests{topic="harmony/sync/mainnet/0/1.0.0"}

# Available streams
hmy_stream_request_manager_num_available_streams{topic="harmony/sync/mainnet/0/1.0.0"}

# Stream utilization
hmy_stream_request_manager_num_available_streams{topic="harmony/sync/mainnet/0/1.0.0"} / hmy_stream_request_manager_num_connected_streams{topic="harmony/sync/mainnet/0/1.0.0"}
```

### Health Check Endpoints

Stream Sync exposes health information through various interfaces:

#### Stream Health Statistics
```go
// Get stream health statistics
stats := requestManager.GetStreamHealthStats()
// Returns: totalStreams, availableStreams, totalFailures, failureDistribution, averageFailures
```

#### Stream Status Monitoring
- **Active Streams**: Real-time count of connected streams
- **Reserved Streams**: Backup stream pool status
- **Request Queues**: Waiting and pending request counts
- **Failure Tracking**: Stream failure counts and distributions
- **Discovery Status**: Active peer discovery operations

## Request Manager

The Request Manager is a critical component of Stream Sync that handles the lifecycle of all sync requests, from submission to completion. It manages request queues, stream allocation, and response delivery.

### How Request Manager Works

#### Request Lifecycle
1. **Request Submission**: New requests are added to the waiting queue
2. **Stream Selection**: Available streams are selected based on health and load
3. **Request Processing**: Requests are sent to selected streams
4. **Response Handling**: Responses are matched with pending requests
5. **Completion**: Requests are marked as complete and removed from tracking

#### Request Queues
The Request Manager maintains multiple priority queues:

- **High Priority Queue**: For urgent requests (e.g., epoch sync, consensus-triggered downloads)
- **Low Priority Queue**: For regular sync requests (e.g., block downloads, state sync)

#### Stream Allocation Strategy
- **Health-Based Selection**: Streams with lower failure counts are prioritized
- **Load Balancing**: Distributes requests across available streams
- **Protocol Compatibility**: Ensures requests match stream capabilities
- **Blacklist/Whitelist**: Supports stream filtering for specific request types

### Request Management Features

#### Request States
- **Waiting**: Request is queued and waiting for an available stream
- **Pending**: Request is actively being processed by a stream
- **Completed**: Request has received a response and is finished
- **Failed**: Request encountered an error or timeout

#### Timeout Management
- **Pending Request Timeout**: 180 seconds for active requests
- **No Stream Timeout**: 60 seconds before canceling waiting requests
- **Progress Monitoring**: Tracks request progress to detect stalls

#### Concurrency Control
- **Write Semaphores**: Limits concurrent writes to streams (max 32)
- **Throttling**: Processes requests in batches (16 per throttle cycle)
- **Stream Limits**: Each stream handles one request at a time

### Request Processing Pipeline

#### 1. Request Submission
```go
// Submit a new sync request
response, streamID, err := requestManager.DoRequest(ctx, syncRequest)
```

#### 2. Queue Management
- Requests are added to appropriate priority queue
- Queue size is limited to 1024 requests
- High-priority requests are processed first

#### 3. Stream Selection
```go
// Stream selection algorithm
func pickAvailableStream(req *request) (*stream, error) {
    // Find eligible streams
    // Sort by failure count (lowest first)
    // Return healthiest available stream
}
```

#### 4. Request Execution
- Request is assigned to selected stream
- Stream availability is marked as false
- Request timeout is set
- Request is added to pending tracking

#### 5. Response Handling
- Responses are delivered through dedicated channel
- Request-response matching is validated
- Completed requests are removed from tracking
- Stream availability is restored

### Error Handling and Recovery

#### Request Failures
- **Stream Write Errors**: Request is marked as failed and stream is marked unavailable
- **Timeout Errors**: Requests are automatically canceled after timeout
- **Stream Removal**: Pending requests are canceled when streams are removed

#### Recovery Mechanisms
- **Automatic Retry**: Failed requests can be retried with different streams
- **Stream Replacement**: Failed streams are replaced from reserved pool
- **Queue Management**: Failed requests don't block the queue

### Performance Optimizations

#### Batch Processing
- **Throttle Intervals**: 100ms intervals for processing waiting requests
- **Batch Sizes**: 16 requests processed per throttle cycle
- **Concurrent Writes**: Up to 32 concurrent stream writes

#### Memory Management
- **Request Pooling**: Reuses request objects to reduce allocations
- **Stream Tracking**: Efficient stream state management
- **Queue Optimization**: Double-linked list for fast insertions/removals

#### Monitoring and Metrics
- **Request Counts**: Waiting, pending, and completed request tracking
- **Stream Utilization**: Available vs. connected stream ratios
- **Performance Histograms**: Request latency and processing time distributions

### Configuration Options

#### Timeout Settings
```go
const (
    throttleInterval = 100 * time.Millisecond
    StreamMonitorInterval = 1000 * time.Millisecond
    NoStreamTimeout = 60 * time.Second
    throttleBatch = 16
    PendingRequestTimeout = 180 * time.Second
    deliverTimeout = 5 * time.Second
    maxWaitingSize = 1024
    writeConcurrency = 32
)
```

#### Queue Management
- **Priority Queues**: Separate queues for high and low priority requests
- **Queue Limits**: Maximum 1024 waiting requests
- **Throttling**: Controlled request processing to prevent overwhelming streams

## Stream Manager

The Stream Manager is responsible for maintaining the pool of active streams, handling stream lifecycle events, and ensuring optimal stream availability for the Request Manager.

### Stream Lifecycle Management

#### Stream States
1. **Active**: Stream is connected and available for requests
2. **Reserved**: Stream is connected but held as backup
3. **Removed**: Stream has been disconnected and is in cooldown
4. **Expired**: Stream has completed cooldown and can reconnect

#### Stream Addition Process
```go
func handleAddStream(st sttypes.Stream) error {
    // Check if stream already exists
    // Validate stream compatibility
    // Add to main streams or reserved pool
    // Update metrics and emit events
}
```

#### Stream Removal Process
```go
func handleRemoveStream(id sttypes.StreamID, reason string, criticalErr bool) error {
    // Remove from active streams
    // Handle trusted peer exceptions
    // Apply cooldown period
    // Trigger replacement from reserved pool
    // Update metrics and emit events
}
```

### Health Monitoring and Watchdogs

#### Stream Health Checks
- **Connection Status**: Monitors underlying network connection health
- **Failure Tracking**: Counts stream failures and tracks failure patterns
- **Progress Monitoring**: Tracks data transfer progress to detect stalls
- **Timeout Detection**: Identifies streams that are unresponsive

#### Health Check Intervals
```go
const (
    checkInterval = 30 * time.Second        // Stream count check interval
    discTimeout = 10 * time.Second          // Discovery timeout
    connectTimeout = 60 * time.Second       // Stream setup timeout
    StreamMonitorInterval = 1000 * time.Millisecond  // Request manager monitoring
)
```

#### Watchdog Mechanisms
- **Stream Count Monitoring**: Automatically triggers discovery when stream count drops
- **Discovery Cooldown**: Prevents excessive discovery attempts
- **Reserved Stream Promotion**: Instantly replaces failed streams from backup pool
- **Trusted Peer Protection**: Special handling for trusted peers

### Discovery and Connection Management

#### Peer Discovery Process
```go
func discoverAndSetupStream(ctx context.Context) (int, error) {
    // Connect to trusted peers first
    // Use DHT discovery for additional peers
    // Limit concurrent connection attempts
    // Apply connection timeouts
    // Handle discovery failures gracefully
}
```

#### Connection Limits
- **Setup Concurrency**: Maximum 16 concurrent stream setup operations
- **Discovery Batching**: Configurable batch sizes for peer discovery
- **Connection Timeouts**: 60-second timeout for stream establishment
- **Discovery Cooldown**: 5-minute cooldown after failed discovery attempts

#### Trusted Peer Handling
- **Priority Connection**: Trusted peers are connected first
- **Exception Handling**: Trusted peers are not removed for critical errors
- **Persistent Connections**: Maintains connections to trusted peers
- **Fallback Strategy**: Uses trusted peers when discovery fails

### Reserved Stream Pool Management

#### Pool Sizing
- **Maximum Size**: Up to 100 reserved streams can be maintained
- **Dynamic Allocation**: Streams are moved between active and reserved pools
- **Instant Promotion**: Reserved streams are immediately available when needed
- **Load Balancing**: Maintains optimal stream counts across pools

#### Pool Operations
```go
func addStreamFromReserved(count int) (int, error) {
    // Promote reserved streams to active
    // Update stream counts and metrics
    // Maintain pool balance
    // Handle promotion failures
}
```

### Cooldown and Recovery Management

#### Cooldown System
- **Progressive Backoff**: Each failure increases cooldown duration
- **Failure Counting**: Tracks consecutive failures per stream
- **Cooldown Periods**: 5 minutes × failure count, max 60 minutes
- **Automatic Recovery**: Streams can reconnect after cooldown expires

#### Recovery Triggers
- **Stream Count Thresholds**: Automatic discovery when below soft/hard caps
- **Reserved Pool Exhaustion**: Discovery triggered when backup streams are depleted
- **Health Degradation**: Discovery triggered when stream health scores drop
- **Manual Recovery**: Administrative commands to force discovery

### Performance Monitoring

#### Stream Metrics
- **Connection Counts**: Active, reserved, and total stream counts
- **Health Scores**: Stream failure rates and health distributions
- **Discovery Rates**: Peer discovery success and failure rates
- **Connection Latency**: Stream setup and establishment times

#### Health Indicators
- **Stream Availability**: Ratio of available to total streams
- **Failure Distribution**: Distribution of stream failure counts
- **Discovery Efficiency**: Success rate of peer discovery operations
- **Recovery Time**: Time to restore optimal stream counts

### Configuration and Tuning

#### Stream Manager Configuration
```go
type Config struct {
    HardLoCap int      // Immediate discovery trigger threshold
    SoftLoCap int      // Routine discovery trigger threshold
    HiCap int          // Maximum stream count
    DiscBatch int      // Discovery batch size
    TrustedPeers map[libp2p_peer.ID]struct{} // Trusted peer list
}
```

#### Tuning Guidelines
- **Stream Caps**: Balance between performance and resource usage
- **Discovery Intervals**: Balance between responsiveness and overhead
- **Cooldown Periods**: Balance between recovery and stability
- **Trusted Peers**: Maintain sufficient trusted peer connections

## Future Enhancements

### Planned Improvements
- **Adaptive Concurrency**: Automatic adjustment based on network conditions
- **Quality-Based Peer Selection**: Prioritize high-quality sync peers
- **Compression**: Reduce bandwidth usage for sync data
- **Incremental Sync**: Only download changed data for faster updates

### Research Areas
- **Cross-Shard Sync**: Optimizing sync across multiple shards
- **State Pruning**: Efficient state management for archival nodes
- **Mobile Optimization**: Light client sync capabilities

## Conclusion

Stream Sync represents a significant improvement in Harmony's network architecture, moving from a centralized, single-point-of-failure system to a decentralized, resilient, and scalable solution. This transition aligns with blockchain's core principles of decentralization while providing better performance and reliability for all network participants.

Validators will experience improved sync performance and contribute to a more robust network infrastructure. The migration is designed to be seamless, with no action required from existing validators.

As Harmony continues to grow, Stream Sync will provide the foundation for a truly decentralized and scalable blockchain network. 
