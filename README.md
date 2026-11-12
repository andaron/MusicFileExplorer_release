# Music File Explorer

A local Windows desktop app for browsing game audio, auditioning edits, and exporting files or regions. The catalog and export settings persist between sessions.

## Download and run

1. Install the [.NET Desktop Runtime 10 for Windows x64](https://dotnet.microsoft.com/en-us/download/dotnet/10.0).
2. Download **MusicFileExplorer-win-x64.7z** under **Assets** and extract the entire archive.
3. Open the extracted `MusicFileExplorer-win-x64` folder and run `MusicFileExplorer.exe`.
## Use

1. Choose **+ Folder** and select one or more library folders. Scanning includes subfolders and skips macOS `._` sidecar files. Use **Roots** to manage scan locations and **Folder tree** to filter by an indexed directory and its descendants. **All folders** clears the directory filter.
2. Type in the search box. All space-separated terms must match somewhere in the filename, path, embedded text metadata, or assigned categories and tags. Terms are case-insensitive substrings. Combine them with the folder, label, format, duration, sample rate, or channel filters. Click a column header to sort the full result set. **Save search** stores the query, filters, and sort order. Highlight an item in **Saved searches** and click **×** to delete it.
3. Select a row for its waveform. Click the waveform to seek, drag across it to mark a region, or double-click it to clear the region. You can also enter seconds or `m:ss.fff` in **In** and **Out** and press Enter. **Play region** plays the marked interval. **Loop** repeats the active file or interval. Clicking outside an active region returns playback to the full file.
4. In **Settings**, set the default export folder, format, and conversion options. The preview panel displays the output path before you click **Export**. **Choose folder** overrides the folder for one successful export. Toggle **Use region** and **Apply pitch / speed** as needed. A conflict offers overwrite, a numbered filename, or cancel. An existing cataloged source is protected from overwrite.
5. Right-click a selected row to retry indexing, export, show in File Explorer, tag, or delete the whole selection. Play and Stop control the preview. Batch export uses full files and numbers conflicting output names. Dragging selected rows to a Windows file-drop target sends the original files with the copy effect.
6. Scanning matches starter categories against words in file and folder names and imports embedded genre and grouping metadata as tags for new or changed files. Existing entries get path-based categories from the catalog during the upgrade. Use **Manage** to edit your categories and tags, or **Tag selected** to change assignments on multiple rows. Manual removals take precedence over automatic assignments.
7. The **Delete** dialog lets you remove catalog entries, permanently delete the source files, or do both. Catalog-only removal keeps the source files available for future scans. Disk-only deletion keeps the catalog entries marked **Missing**.

Drag the dividers to resize the sidebar, folder and tag lists, or waveform panel. Use the page arrows or a page number to navigate the results. Double-click a slider to reset it, or click its value to type a number. Press **Enter** or **Space** to apply the value within the slider’s range. Toggle **Auto play** next to **Loop** to play clicked files and files selected with Previous or Next. **Show folder in tree** in a file’s right-click menu reveals its directory and filters the results to that folder. The status bar collapses after 10 seconds without a new message and stays visible during active operations.

| Shortcut | Action |
| --- | --- |
| Ctrl+F | Focus library search |
| Space | Play or pause the marked region, or the full file when no region is marked |
| Ctrl+Space | Start the full file from the beginning |
| Esc | Stop preview |
| Alt+Left / Alt+Right | Previous or next result |
| R | Play the marked region |
| L | Toggle loop |
| M | Toggle mute |
| Ctrl+E | Export |

## Formats and processing

| Operation | Formats and options |
| --- | --- |
| Index, waveform, and preview | WAV, RF64, Wave64, MP3, FLAC, Ogg Vorbis, Ogg Opus, AIFF/AIFC, AU, and CAF when the decoder supports the file's encoding |
| WAV and AIFF export | 16-bit PCM, 24-bit PCM, or 32-bit float |
| FLAC export | 16-bit or 24-bit PCM |
| OGG export | Ogg Vorbis |
| Original export | Byte-for-byte copy of the full file, including its metadata |

Encoded exports can keep the source sample rate and channels or convert to a supported rate and mono or stereo. Pitch is adjustable from −12 to +12 semitones. Speed ranges from 0.5× to 2× and preserves pitch. Preview volume and mute affect playback only. Encoded exports preserve text metadata supported by the source decoder and destination encoder.

## Storage and architecture

- `%LOCALAPPDATA%\MusicFileExplorer\catalog.db` holds the SQLite catalog, folder list, labels, saved searches, and export settings. The app also uses SQLite WAL and shared-memory files beside it while running. Removing a folder hides entries outside the remaining library roots and retains their labels if you add the folder again.
- `Waveforms\` holds cached min/max peaks keyed by the path, file size, and modification timestamp. `diagnostics.log` and its rotated predecessor record scan and app errors without audio samples. Set `MUSIC_FILE_EXPLORER_DATA_DIR` to an absolute folder before launching to use an isolated catalog for testing.
- The WPF view model coordinates cancellable background operations. SQLite FTS5 indexes partial search terms and the library grid pages through results with a virtualized view. Scans update the catalog in small transactions and reuse unchanged metadata. Folder overlaps use a single path-based file record.
- Preview and export share a streaming region, channel, pitch/speed, and resampling pipeline. Playback and decoder operations run on worker threads. The waveform pass only runs for selected files. Exports write a temporary file beside the destination and rename it after encoding completes.

## Dependencies

| Dependency | Role and license |
| --- | --- |
| [NAudio 3.3](https://github.com/naudio/NAudio) | .NET 9+ libraries compatible with .NET 10. The app references WASAPI playback and SoundFile streaming, plus Core through those packages. MIT. |
| [libsndfile 1.2.2](https://github.com/libsndfile/libsndfile/releases/tag/1.2.2) | Replaceable native x64 codec DLL for the supported input and output formats. LGPL-2.1-or-later. |
| [SoundTouch.Net 2.3.2](https://www.nuget.org/packages/SoundTouch.Net/2.3.2) | Managed .NET Standard DSP for independent pitch shift and tempo changes. LGPL-2.1-or-later. |
| [TagLibSharp 2.3.0](https://www.nuget.org/packages/TagLibSharp/2.3.0) | Managed .NET Standard fallback for searchable embedded tags and source bit depth. LGPL-2.1-only. |
| [Microsoft.Data.Sqlite 10.0.15](https://www.nuget.org/packages/Microsoft.Data.Sqlite/10.0.15) | MIT-licensed SQLite access. SQLitePCLRaw brings the SQLite runtime, which contains public-domain SQLite. |

For redistribution, keep the third-party DLLs replaceable and include the applicable license notices and corresponding-source information for the LGPL components.

## Known limitations

- Rescan to discover filesystem changes. A move or rename appears as a missing entry and a new path. Tags stay on the old catalog entry. Directory junctions and symlinks encountered inside a root are skipped to avoid loops.
- Missing bit-depth values are shown as a dash. Lossy codecs generally have no source PCM bit depth. Encoded exports may omit artwork, cue points, and proprietary metadata.
- Preview uses stereo and the default Windows output device. Downmixing uses channel averages. For sources with more than two channels, choose mono or stereo when exporting with pitch or speed changes. The playhead is an estimate when stretching or looping.
- Unsupported or damaged files are marked unreadable during indexing. Select the affected rows and choose **Retry indexing** to refresh just those entries. If a file becomes unavailable after a scan, preview or export shows the immediate error.
- The app uses libsndfile's format support. AAC/M4A and WMA are outside the indexed formats. WAV export uses standard WAV size limits.
