# jquery-selectSuggest
Creates a simple bootstrap dropdown for large datasets from the server.

> This plugin was developed for single selection only for now.
### Requirements
- Boostrap >= 5.0
- jQuery 3.6 -> I can't name the minimum jQuery version, I developed it on jQuery 3.6.
### Installation
Simply include the following script at the end of the body tag.
```html
<script src="jquery.bsSelectSuggest.js"></script>
```
No further CSS needed, the current bootstrap classes are used.

### Usage
##### html
Place the hidden input field where you want the dropdown to appear.
```html
<input type="hidden"
       id="exampleInput"
       data-bs-toggle="suggest"
       data-bs-target="path/to/actions.php"
>
```
##### javascript
```js
$('#exampleInput').each(function(i, input){
    $(input).suggest(options||{});
});
```
> Please make a separate call for each suggestion box!
### Options
The following options are currently implemented.
```json
{
    "limit": 10, // the maximum number of records
    "darkMenu": false, // show the dropdown in dark style
    "btnClass": "btn btn-outline-secondary", // dropdown Button class
    "emptyText": "Bitte wählen.." // placeholder for no selection 
}
```
### Methods
Currently, there is only one method.
```js
$('selector').suggest('val', value);
```
### Events
Currently, there is only one event
```js
$('selector')
    .on('suggest-change', function(e, id, text){
        // id: the item id
        // text: the item text
        console.log(id, text, 'was selected');
    })
```
### Required response for suggestion
The parameters `q` and `limit` are sent to the server via `GET`.
`q` is in this case the search string and `limit` the maximum number of records to be determined.
As response the plugin expects an `array` with `items` and the `total` number of records.  
An item consists of the attributes `id` and `text`.
```json
{
    "items": [{
        "id": 1,
        "text": "Germany"
      },{
        "id": 2,
        "text": "Spain"
      },{
        "id": 3,
        "text": "Italy"
      }
    ],
    "total": 75
}
```
When the method `val` is called, only the parameter `value` is sent to the server   
and only **one item object** is expected (no array).
```js
// call method val
$('selector').suggest('val', value);

// response
{
    "id": 1,
    "text": "Germany"
}
```

### backend example
A complete example can be found in the demo folder.
```php
<?php
header('Content-Type: application/json');

try {
    // Fetch a test data set
    /** @var stdClass[] $countries */
    $countries = json_decode(file_get_contents('countries.json'), false, 512, JSON_THROW_ON_ERROR);

    // Try to find the query parameter value
    $value = filter_input(INPUT_GET, 'value', FILTER_VALIDATE_INT);

    /** @var null|stdClass|array $return */
    $return = null;

    // Was the value parameter found?
    $fetchSingleData = ! empty($value);

    // if yes
    if ($fetchSingleData)
    {
        // Get the record using the value parameter
        $data = array_filter($countries, static function($country) use ($value){
            return $country->id === $value;
        });
        $return = $data[0];
    }
    // if no
    else
    {
        // Get parameter q and limit
        $limit = filter_input(INPUT_GET, 'limit', FILTER_VALIDATE_INT) || 10;
        $q = filter_input(INPUT_GET, 'q');
        $search = empty($q)? false : strtolower($q);

        // If q was not passed or is empty, do not return any results either.
        // Otherwise, search for matches of the search string.
        $data = $search === false ? [] :  array_slice(
            array:array_filter($countries, static function($country) use ($search){
                return str_contains(strtolower($country->text), $search);
            }),
            offset: 0,
            length: $limit
        );

        // Put the result in the response
        $return['items'] = $data;
        $return['total'] = count($countries);
    }
    // Return as JSON
    http_response_code(200);
    exit(json_encode($return, JSON_THROW_ON_ERROR));
} catch (JsonException $e) {
    http_response_code(500);
    exit($e->getMessage());
}

```