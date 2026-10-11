The `query` tag performs a query on a dataset. The dataset must already exist: create it first with the [create](tag-create.md) tag. By default the tag body is read as the **path of a query file**; to write the query (SQL, SPARQL or an ASP program) inline in the body, set `isfilepath=False`. Without it, an inline query fails with an error such as `File name too long` or `No such file or directory`.

What the tag returns depends on the data backend:

| `datastore` | Query language | Returns |
|-------------|----------------|---------|
| `duckdb`    | SQL            | a Pandas DataFrame, one column per selected column |
| `rdflib`    | SPARQL         | a Pandas DataFrame, one column per SELECT variable (IRIs as `<http://...>`, literals as plain strings) |
| `clingo`    | ASP rules      | a **list** with one entry per answer set (see below) |

For `clingo`, the rules in the tag body are added to the dataset's facts and solved; every answer set becomes one list entry, in the format chosen by `rformat`:

- `rformat="lp"` (default): a string of facts, e.g. `"grandparent(tom,ann).\n"`
- `rformat="dict"`: a dict mapping each predicate to a list of argument lists, e.g. `{"grandparent": [["tom", "ann"]]}`
- `rformat="df"`: a dict mapping each predicate to a DataFrame with columns `arg1`, `arg2`, ...

Arguments are returned as strings. In ASP a bare constant (`sam`) and a quoted string (`"sam"`) are different terms: write rules with the same form the facts use (`taken(sam,C)` matches the fact `taken(sam,intro)`, `taken("sam",C)` does not). Quoted strings come back with their quotes (`"tom"` is returned as `'"tom"'`). Use `predicate` to keep only some predicates. A program with exactly one answer set (e.g. pure Datalog-style rules) returns a list of length 1, so use `res[0]`.

To loop over the rows of a DataFrame result, prefer `res.to_dict("records")`, which keeps each column's type. `res.iterrows()` turns every row into a single-type Series, so when all selected columns are numeric, integers come out as floats (`1` prints as `1.0`); add `| int` where needed if you use it.

Here are parameters of the `query` tag:

| Name          | Description                                                    |
|---------------|----------------------------------------------------------------|
|`dataset`      | Name of a dataset to query (default `kb`)               |
|`datastore`    | Data backend. `clingo`, `duckdb`, and `rdflib` are available. (by default, `rdflib`) |
|`isfilepath`   | A bool value to denote if the given data is a file path or not (default `True`, which denotes the given data is a file path) |
|`predicate`    | Name of the predicate to be queried (default `None`). Available for `clingo` data backend only. |
|`programname`  | Name of the program (default `'base'`). Available for `clingo` data backend only. |
|`rformat`      | Format of the returned data, i.e., `'lp'`, `'df'`, or `'dict'` (default `'lp'`). Available for `clingo` data backend only. |

??? example "Example 1: the given string is not a file path"

    ```
    {% query "test", datastore="rdflib", isfilepath=False %}
        SELECT ?s ?p ?o
        WHERE {
            ?s ?p ?o
        }
        ORDER BY ?s ?p ?o
    {% endquery %}
    ```

??? example "Example 2: the given string is a file path"

    ```
    {% query "test", datastore="rdflib", isfilepath=True %} query_file {% endquery %}
    ```
    Here is the query_file's content:
    ```
    SELECT ?s ?p ?o
    WHERE {
        ?s ?p ?o
    }
    ORDER BY ?s ?p ?o
    ```

??? example "Example 3: an ASP query with clingo"

    ```
    {% create "family", datastore="clingo", inputformat="lp", isfilepath=False %}
        parent(tom,bob). parent(bob,ann).
    {% endcreate %}
    {% query "family", datastore="clingo", rformat="dict", predicate="grandparent", isfilepath=False as res %}
        grandparent(X,Z) :- parent(X,Y), parent(Y,Z).
    {% endquery %}
    {% for args in res[0]["grandparent"] | sort %}
    {{ args[0] }} is a grandparent of {{ args[1] }}
    {% endfor %}
    {% destroy "family", datastore="clingo" %}
    ```

    Expected output:
    ```
    tom is a grandparent of ann
    ```

    With several answer sets (e.g. choice rules), loop over them: `{% for model in res %} ... {% endfor %}`.
