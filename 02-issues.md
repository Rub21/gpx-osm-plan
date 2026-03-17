# GPS Traces — Known Issues

From the [proposal document](https://hackmd.io/@1ec5/Hkt6a3P8Ze)

---

## Issues

### Database / Backups

The `gps_points` table has ~31.8 billion rows as of February 2026 (was 22 billion in 2023 — [source](https://x.com/OSM_Tech/status/1684295081199562752)). This table is about ~ 1.2 TB, which makes database backups very large and slow. Stats: https://planet.openstreetmap.org/statistics/data_stats.html

The system also saves the complete original GPX file uploaded by the user, even if it contains private information (like the user's home folder path in the file metadata).

### Rails App

**Pagination:** In 2023, the API for listing traces (`/api/0.6/gpx_files`) was migrated from page numbers to trace IDs ([PR #4089](https://github.com/openstreetmap/openstreetmap-website/pull/4089)). But the trackpoints endpoint (`/api/0.6/trackpoints`) was not migrated and still uses page-number pagination with OFFSET. High page numbers cause timeouts because PostgreSQL has to scan and skip all previous rows. JOSM calls this endpoint in the background and silently ignores the errors — users don't see the failures. API docs: https://wiki.openstreetmap.org/wiki/API_v0.6#GPS_traces

**Massive uploads:** Telemetry systems have been uploading large amounts of traces, causing delays in the import queue for normal users. See [community discussion](https://community.openstreetmap.org/t/gps-trace-upload-delay/104881).

**GIF generation:** The animated GIF generation in the Rails app depends on libgd2, an old library. The Ruby wrapper for it ([gd2-ffij](https://github.com/dark-panda/gd2-ffij)) has an open PR without maintenance: [gd2-ffij PR #28](https://github.com/dark-panda/gd2-ffij/pull/28).

### Tile Renderer

**Cannot delete traces:** If a user deletes or redacts a trace, the tile renderer cannot remove it from the existing tiles. In 2023, the team had to rebuild the entire tileset after deleting some traces — this process took about one month. See [gpx-updater issue #4](https://github.com/openstreetmap/gpx-updater/issues/4).

**No active maintainers:** The tile rendering pipeline ([gpx-updater](https://github.com/openstreetmap/gpx-updater/), [gpx-import](https://github.com/e-n-f/gpx-import/), [datamaps](https://github.com/e-n-f/datamaps/)) is all C code with no one actively maintaining it. Finding someone to fix bugs or add features is difficult.


