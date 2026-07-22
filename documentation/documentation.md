# Duplicate Record Audit Report

### playlists table

The playlist table has two entries named `"Music" (playlist_id 1 and 8)` and two named `"TV Shows" (playlist_id 3 and 10)`.  
On its own, playlist only stores a name, it doesn't show what songs are in each playlist, so a matching name alone isn't proof of a real duplicate.

To confirm it, I checked the playlist_track table instead, this is where each song's membership in a playlist is recorded; one row per song per playlist, linked back to playlist through the shared playlist_id column.

First, I compared how many songs each playlist contained. `"Music" (id 1)` and `"Music" (id 8)` both had exactly `3,290 songs.`  
`"TV Shows" (id 3)` and `(id 10)` both had exactly `213.`  
Matching counts were a strong hint, but not final proof, two playlists could have the same number of songs without containing the same actual songs.

To confirm they held the exact same songs, I used `EXCEPT`, which compares two lists and returns anything present in the first list but missing from the second:

```sql
SELECT track_id FROM playlist_track WHERE playlist_id = 1
EXCEPT
SELECT track_id FROM playlist_track WHERE playlist_id = 8;
```

This returned zero rows for both playlist pairs, meaning no song existed in one playlist that was missing from the other. Combined with the matching song counts, this confirms `"Music" (1, 8)` and `"TV Shows" (3, 10)` are true duplicates, not a coincidence of naming.

Two other same-named pairs, `"Movies" (id 2, 7)` and `"Audiobooks" (id 4, 6)`, were left unconfirmed, since both are empty `(0 songs)`, there isn't enough data to prove or disprove duplication either way.

```

```
