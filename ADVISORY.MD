# Advisory

## Shortest Path Vulnerability

Authenticated clients could request a calculated route from a start point to an end point with an arbitrary number of inner points by POSTing a json-encoded path request to `/api/path`:

```json
{
  "start": {
    "x": "fooX",
    "y": "fooY"
  },
  "sub": [
    "1337",
    "1338",
    "1339",
    "1340",
    "1341"
  ],
  "finish": {
    "x": "barX",
    "y": "barY"
  }
}
```

The server would now read the points mentioned in `sub` without considering whether they were not public and not belonging to the user sending the request: The response would contain the X and Y coordinates of every point whose id was supplied in `sub`.

```c#
var inner = request.Inner?
  .Select(PointHolder.GetPoint)
  .WhereNotNull()
  .Select(point => new DrawPoint {X = point.X, Y = point.Y})
  .Distinct()
  .ToList();
```

This LINQ query takes every string in sub, projects it to the result of GetPoint (which queries the `Point` from the storage), drops nulls, and projects each to a `DrawPoint` with the same X and Y values. The resulting list is used to process a nice route which is sent to the client.

Since the flagbot did never bother to use this feature, our fix just [aborted computation](https://github.com/ENOFLAG/ructfe2017_TreasureMap/commit/0724015f53ff0f5aa310dbc7bb0b956095790d04) if X and Y contained a flag.
