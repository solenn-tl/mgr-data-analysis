# Architecture
## 1. System architecture & performance design
When dealing with large text datasets in web applications—such as the alignment table (merged_sentences.csv) used by this platform, which contains ~450,000 rows (~74 MB)—traditional file loading architectures run into severe bottlenecks:

* Single-Threaded Network Transfer Lag: Standard fetch() or XMLHttpRequest downloads the entire file into a buffer before handing it to JavaScript. Using basic local servers (e.g., Python's single-threaded http.server), this creates a prolonged network lockup phase.

* The Main-Thread Boundary Crash: Passing a parsed array of half a million objects from a standard Web Worker or parsing it all at once on the main execution thread triggers massive garbage collection cycles, causing the browser window to freeze completely ("Thread Lock").

To bypass these limits, this application uses a custom Asynchronous Chunk-Streaming Architecture:

```
[ Local HTTP Server ]
         │
         │ (Iterative Binary Data Transfer)
         ▼
 1. Native Fetch Stream Reader ──► [ Live Percentage UI Updates ]
         │
         │ (Accumulated Binary Arrays)
         ▼
 2. Zero-Copy Binary Memory Blob
         │
         │ (Streamed in Chunks of ~20,000 Rows)
         ▼
 3. PapaParse Chunking Loop ──► [ Live Row Ingestion Counter ]
         │
         │ (Direct Mutation to O(1) Dictionary Index)
         ▼
 4. ShallowRef State Lock ──► UI Hydrated & Fully Responsive
```
### Key Technical Pillars:
1. **Native Stream Ingestion Protocol**: The application reads data straight from the network socket using the browser's native fetch().body.getReader(). Instead of waiting for the full 74 MB file to load, it processes incoming data packets immediately. This allows the UI to show an explicit download progress indicator (Downloading alignment table: 34.2 MB / 74.0 MB (46%)).
2. **Zero-Copy Memory Blobs**: Once the data packets arrive, they are wrapped inside an immutable in-memory binary Blob. This structure bypasses JavaScript text parsing overhead and allows data loading speeds that mimic manual local drag-and-drop actions.
3. **Non-Blocking Chunk Parsing**: The binary Blob is fed into a streaming PapaParse configuration using the chunk callback interface. The data is parsed in increments of roughly 20,000 rows at a time. The event loop yields to the browser between chunks, keeping the UI fully interactive and rendering a rolling text index counter (Ingesting corpus: 240,000 segments indexed...) instead of freezing.
4. **Optimized Hash-Map Data Indexing ($O(1)$ Complexity)**: Instead of saving rows into a flat array that requires filtering via slow, recurring .filter() iterations, rows are directly compiled into a hash-indexed lookup table keyed by recording_id. This reduces sub-segment queries from $O(N)$ linear scans down to $O(1)$ constant time lookups:

$$\text{Indexed Structure: } \{\text{ "recording\_id\_key" } : [ \text{Row}_1, \text{Row}_2, \dots, \text{Row}_k ] \}$$

5. **Shallow Reactivity Lock**: The global datasets are wrapped in Vue 3 shallowRef wrappers rather than standard deep ref wrappers. This tells Vue to track only the top-level pointer rather than building deep reactive dependencies for all 450,000 objects, saving massive amounts of memory.

## 2. Core Functional FeaturesAutomated 

1. **Hot-Linked Synchronization**: On initial launch, the pipeline automatically detects and ingests merged_no_transcriptions.json and merged_sentences.csv straight from the root directory.
2. **Zero-Config Manual Fallback**: If the app is run from a local file path (file://) where browser security constraints block network streams, it gracefully falls back to dual manual file upload slots.
3. **Multi-Tiered Hierarchical Traversal**: Users can browse high-level parent records in a card interface, filter by metadata properties, and drill down into sub-segment lists with synchronized media player states.
4. **Dynamic Structural View Window Adjustments**: Supports full parent-timeline scanning or sub-segment locking. When limited to a specific sub-segment, the player isolates the audio file within the time constraints:$$\text{Playback Window Constraint: } [t_{\text{cin}}, t_{\text{cout}}]$$If the timeline crosses $t_{\text{cout}}$, the media engine automatically pauses and loops back to $t_{\text{cin}}$.
5. **Anomalous Discourse Filters**: The interface includes live diagnostic analytics that filter sentences into specific linguistic and technical categories:Hallucinations: Tracks processing anomalies (is_hallu === true).
6. **Repetitions**: Flags loops in speech delivery patterns (is_repetition === true).Empty or Punctuation Only: Pinpoints unvocalized silent segments (is_empty_or_punctuation_only === true).
7. **Invalid Segments**: Displays segments that failed verification tests (valide === false).
8. **SRT Subtitle Export Engine**: Dynamically transforms the active filtered timeline view into standard, cross-platform Synchronized Subtitle tracks (.SRT) complete with accurate millisecond time tracking and speaker identification tags.

## 3. Data Schema Expectations
To load successfully, the pipeline expects two data files to reside in the same directory as the HTML application file:

File 1: ```merged_no_transcriptions.json```
Contains structural metadata arrays describing conferences, creators, dates, and nested segment parameters (children).
```json
{
  "data": [
    {
      "id": 2027,
      "type": "video",
      "notice_type": "MPV1",
      "title": "Musique et numérique",
      "subtitle": "le 28 mars 2017",
      "date": "2017-03-28T00:00:00.000Z",
      "stream": "https://...",
      "file": "https://...",
      "duration": 3749.1,
      "children": []
    }
  ]
}
```

File 2: ```merged_sentences.csv```
Contains individual sentence alignments that map back to the parent structures using reference indices.
```csv
recording_id,transcription_id,segment_index,sentence,sentence_start,sentence_end,speaker,is_empty_or_punctuation_only,is_hallu,is_repetition,valide
3,2816,0,Alors je crois qu'en quelques minutes...,0.031,3.156,SPEAKER_00,False,False,False,True
```