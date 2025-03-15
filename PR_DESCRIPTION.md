# Implement CONTAINS function for DuckDB

This PR implements the CONTAINS function for DuckDB in the legend-engine repository. The implementation leverages the native DuckDB CONTAINS function which returns true if a search_string is found within a string, and adds support for collection CONTAINS operations using the IN operator.

## Implementation Details
- Added the string 'contains' function implementation:
  ```
  dynaFnToSql('contains', $allStates, ^ToSql(format='contains(%s, %s)', transform={p:String[2]|$p}))
  ```

- Added the collection 'in' function implementation:
  ```
  dynaFnToSql('in', $allStates, ^ToSql(format='%s in %s', transform={p:String[2] | if($p->at(1)->startsWith('(') && $p->at(1)->endsWith(')'), | $p, | [$p->at(0), ('(' + $p->at(1) + ')')])}))
  ```

This implementation follows the same pattern as other database implementations and leverages DuckDB's native capabilities. The changes are minimal and focused on adding the necessary function translations to support the CONTAINS operation in DuckDB.

Link to Devin run: https://app.devin.ai/sessions/2c3220b9df8c476f93d6fe5c376f0c3f

Requested by: rajiv.dudhia@gs.com
