The `create` tag creates a dataset based on the given string. By default, the given string is a file path. However, it can be updated by setting the `isfilepath` field to False. Here are parameters of the `create` tag:

|Name           | Description |
|---------------|-------------|
|`dataset`      |Name of RDF dataset to create (by default, `kb`) |
|`datastore`    |Data backend. `clingo`, `duckdb`, and `rdflib` are available. (by default, `rdflib`) |
|`inputformat`  |Format of the input data (by default, `json-ld`, which only suits `rdflib`, so set it explicitly for other backends). `rdflib`: one of {`xml`, `n3`, `turtle`, `nt`, `pretty-xml`, `trix`, `trig`, `nquads`, `json-ld`, `hext`, `csv`}. `duckdb`: `csv` or `json`. `clingo`: `lp`, `csv`, or `json`. |
|`infer`        |Inference to perform on update choosing from {`none`, `rdfs`, `owl`, `rdfs_owl`} (by default, `none`). Please check [OWL-RL](https://owl-rl.readthedocs.io/en/latest/owlrl.html) document for detailed information. |
|`isfilepath`   |A bool value to denote if the given data is a file path or not (by default: `True`, which denotes the given data is a file path) |
|`table`        |Table name (by default, `df`). Available for `duckdb` data backend only. To put several tables in one DuckDB dataset, use one `create` per table with the same dataset name and different `table` names. |
|`colnames`     |Column names of triples with the format of `[[subject1, predicate1, object1], [subject2, predicate2, object2], ...]` when the input format is csv (by default, `None`). Available for `rdflib` data backend only. |
|`predicate`    |`'isfirstcol'` for using the first column as the predicate name; strings other than `'isfirstcol'` are used as the predicate name directly (by default, `'isfirstcol'`). Available for `clingo` data backend only. |
|`name`         |Name of the program (by default, `'base'`). Available for `clingo` data backend only. |

??? example "Example 1: the given string is not a file path"

    ```
    {% create "test", datastore="rdflib", inputformat="nt", isfilepath=False %}
        <http://example.com/drewp> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://xmlns.com/foaf/0.1/Person> .
        <http://example.com/drewp> <http://example.com/says> "Hello World" .
    {% endcreate %}
    ```

??? example "Example 2: the given string is a file path"

    Here is the `test.nt` file:

    ```file
    <http://example.com/drewp> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://xmlns.com/foaf/0.1/Person> .
    <http://example.com/drewp> <http://example.com/says> "Hello World" .

    ```
    
    Code:
    ```
    {% create "test", datastore="rdflib", inputformat="nt", isfilepath=True %} test.nt {% endcreate %}
    ```

??? example "Example 3: load CSV files into DuckDB tables"

    ```
    {% create "school", datastore="duckdb", inputformat="csv", table="students" %} students.csv {% endcreate %}
    {% create "school", datastore="duckdb", inputformat="csv", table="courses" %} courses.csv {% endcreate %}
    {% query "school", datastore="duckdb", isfilepath=False as res %}
        SELECT s.sname, c.title FROM students s JOIN courses c ON s.cid = c.cid ORDER BY s.sname
    {% endquery %}
    {% for _, row in res.iterrows() %}
    {{ row["sname"] }}: {{ row["title"] }}
    {% endfor %}
    {% destroy "school", datastore="duckdb" %}
    ```

    CSV data is read with Pandas, so an integer column with empty cells becomes a float column (`1.0`) and empty cells become `NaN` rather than SQL `NULL`. Test for missing values with `isnan(col)` instead of `col IS NULL`, and convert ids with `CAST(col AS BIGINT)` only after filtering out `NaN`.

??? example "Example 4: load a Turtle file and ASP facts"

    ```
    {% create "museum", datastore="rdflib", inputformat="turtle" %} artworks.ttl {% endcreate %}
    {% create "family", datastore="clingo", inputformat="lp" %} family.lp {% endcreate %}
    ```
