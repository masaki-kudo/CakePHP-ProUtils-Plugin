# ProUtils Plugin for CakePHP 2.1 #

The ProUtils plugin is a growing collection of useful components, behaviors, helpers, and libraries.

> Note: This CakePHP plugin is designed to work with CakePHP 2.1, this requirement is dictated by the implementation
> of CakePHP 2.1's Events System.  Most components should work with CakePHP 2.0 with little or no modifications. 
>The plugin is not backwards compatiable with CakePHP 1.3 or 1.2 and never will be.

## Behaviors

* CsvExport           - adds the ability to export csv data from the model.
* CustomSluggable     - dynamic slugs generated from multiple fieldsx
* EventDispatcher     - adds CakeEvent dispatchers for all common Model callbacks.
* Revisionable        - maintains a versioned copy of each record change in another table.

### CsvExportBehavior ###

Adds the exportCSV method to the model, exports all records.

You can configure the CsvExport behavior using these options:

* delimiter - The delimiter for the values, default is ;
* enclosure - The enclosure, default is "
* encoding - utf8 or default encoding, default is utf8
* max_execution_time - Extend the execution time for larger datasets, default is 360

The main method of this behavior is
    
    $csvdata = $this->Model->exportCSV();

Enable within your model
    
    public $actsAs = array( 'ProUtils.CsvExport' );

Export data and present as a file download from your controller

    function export() {
        $this->autoRender = false;
        $modelClass = $this->modelClass;
        $this->response->type('Content-Type: text/csv');
        $this->response->download( strtolower( Inflector::pluralize( $modelClass ) ) . '.csv' );
        $this->response->body( $this->$modelClass->exportCSV() );
    }

### CustomSluggableBehavior ###

Dynamic slugs using a combination of fields, additionally you can
use fields from associated model data, or some attributes from the Model object.  

'label' is constructed using variables like... 
    
    '{$name}' or '{$id}' or '{$User.first_name} {$User.last_name}'.

Example Usage within your model
    
```php
public $actsAs = array( 'ProUtils.CustomSluggable'=>array(
    'label'  => '{$name} {$Model::table}',
    'slug' => 'slug', //database field to store the slug
    'separator' => '_',
    'length'=>255,
    'unique'=>true,
    'update'=>true
) );
```
                                 
If you Users table looks like...

    id, first_name, last_name, username, password, 'state', 'country'
    1,Sam, White, swhite, ****, TX, USA

Example of using variables in 'label' and slug generated
    
    '{$username}'                                     // swhite
    '{$id} {$username}'                               // on insert swhite, on update 1-swhite
    '{$first_name} {$last_name}'                      // sam-white  
    '{$first_name} {$last_name} {$state} {$country}'  // sam-white-tx-usa  

In addition to using model data in the slug, the following variables are mapped to
the Model instance attributes.

    {$Model::name}
    {$Model::alias}
    {$Model::displayField}
    {$Model::primaryKey}
    {$Model::name}
    {$Model::table}
    {$Model::tablePrefix}

CustomSluggableBehavior acts like most other Sluggable Behaviors and is based on 
SluggableBehavior in the cakedc/utils repo.

### EventDispatcherBehavior ###

Uses the CakePHP 2.1 Events System to dispatch events for all of the
common Model callbacks.
 
Event listeners that register for an Event will have access to the
$Model instance as the $Event->subject(), and some callbacks also 
present addition arguments dictated by the specific callback. Read
more about CakePHP 2.1's Events System [here](http://book.cakephp.org/2.0/en/core-libraries/events.html)

Config parameters can disable certain events alltogether, or change the
behavior of the Model when an Event is canceled by a listener. 

You can configure the EventDispatcher behavior using these options:

* beforeFind
* afterFind
* beforeSave
* afterSave
* beforeDelete
* afterDelete
* beforeValidate

Each of the above can be configured with three options

* disable - Turn on/off Event dispatching for a specific callback, default false.
* eventName - The event name dispatched, default Model.ModelName.callbackName.
* onPropagationStop - Defines how a stopped event will be handled, default varies, see example below.
  * 'continue' - on Event stop, discontinue processing further listeners but does not abort the callback.
  * 'abort' - on Event stop, discontinue processing further listeners, and aborts the callback.
  * 'die' - on Event stop die()
  * 'exception' - on Event stop throw an exception

Example Usage within your model
    
```php
public $actsAs = array( 'ProUtils.EventDispatcher' );
```

Example Usage with all default configuration values

```php
public $actsAs = array( 'ProUtils.EventDispatcher'=>array(
    'beforeFind' => array(
        'disable'=>false,
        'eventName'=>'Model.{alias}.beforeFind',
        'onStopPropagation'=>'continue'
    ),
    'afterFind' => array(
        'disable'=>false,
        'eventName'=>'Model.{alias}.afterFind',
        'onStopPropagation'=>'continue'
    ),
    'beforeSave' => array(
        'disable'=>false,
        'eventName'=>'Model.{alias}.beforeSave',
        'onStopPropagation'=>'abort'
    ),
    'afterSave' => array(
        'disable'=>false,
        'eventName'=>'Model.{alias}.afterSave',
        'onStopPropagation'=>'continue'
    ),
    'beforeDelete' => array(
        'disable'=>false,
        'eventName'=>'Model.{alias}.beforeDelete',
        'onStopPropagation'=>'abort'
    ),
    'afterDelete' => array(
        'disable'=>false,
        'eventName'=>'Model.{alias}.afterDelete',
        'onStopPropagation'=>'continue'
    ),
    'beforeValidate' => array(
        'disable'=>false,
        'eventName'=>'Model.{alias}.beforeValidate',
        'onStopPropagation'=>'abort'
    )
));
```
    
Events Fired by the EventDispatcher behavior          

* Model.ModelName.beforeFind
* Model.ModelName.afterFind
* Model.ModelName.beforeSave
* Model.ModelName.afterSave
* Model.ModelName.beforeDelete
* Model.ModelName.afterDelete
* Model.ModelName.beforeValidate

### RevisionableBehavior ###

> Currently under developement!, RevisionableBehavior is not aware of multi record inserts, conside this unstable code for now.

Example Usage within your model
    
```php
public $actsAs = array( 'ProUtils.Revisionable' );
```

You will need to create an adjacent Model and table to use this behavior.  So
if you wanted to track revision changes to the Model User, create the Model
UserRevision and the table user_revisions.  The user_revisions table should contain a user_id
column and for performance this column should be indexed.  Another example,
for Model Post, create the Model PostRevision and the table post_revisions with the
int column post_id (indexed).

I plan to add the following features to Revisionable in the near future.

* Finder method(s) for getting at the revisions data
* Rollback method(s) to revert current record to an older revision
* Diff method(s) to compare revisions
* Revision limit, keep x number of past revisions, keep revisions less than x days old
* Save fields, the option to only copy certain columns into revisions
* Trigger fields, only make a revision if certain fields are changed
* CakePHP 2.1 event dispatchers so that other plugins can hook this Behavior

## Libraries ##

* ProTemplateCompiler       - Compile a document from a template and data array.

### ProTemplateCompiler ###

This class can be used anywhere within CakePHP, the syntax for a variable is 
{$myVariable} where array('myVariable'=>'foo') is the value that will replace it. 

Example Usage

```php
App::uses('ProTemplateCompiler', 'ProUtils.Lib');

$template = 'Hello {$User.first_name} {$User.last_name}, this is an example of ProTemplateCompile.';
$data = array( 'User'=> array( 'first_name'=>'John', 'last_name'=>'Doe' ) ;    
$Compiler = new ProTemplateCompiler( $template );

echo $Compiler->compile( $data );
// Output: Hello John Doe, this is an example of ProTemplateCompile.
``` 
## Installation ##

    sudo git clone https://github.com/pronique/CakePHP-ProUtils app/Plugin/ProUtils
    
    //Enable plugin in app/Config/bootstrap.php
    CakePlugin::load('ProUtils');
    
## Requirements ##

* PHP version: PHP 5.3+
* CakePHP version: 2.1

## License ##

Copyright 2011-2012, [PRONIQUE Software](http://pronique.com)

Licensed under [The MIT License](http://www.opensource.org/licenses/mit-license.php)<br/>
Redistributions of files must retain the above copyright notice.

--------------------------------------------------------------------------
ProUtils is Open Source Software created and managed by PRONIQUE Software.

http://www.pronique.com
