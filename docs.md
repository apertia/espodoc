Here is a merged coding documentation for usage in an AI prompt, limited to approximately 50,000 tokens:

# EspoCRM Coding Documentation

## Class Naming Conventions

- Use `Espo` namespace for core classes.
- Use camelCase for class names (e.g. `EmailTemplate`, `EmailSender`).
- Use suffix "Factory" for factory classes (e.g. `CollectionFactory`, `SelectBuilderFactory`).
- Use suffix "Manager" for manager classes (e.g. `AclManager`, `FieldManager`).

## Directory Structure

- `application/Espo/` - core files
  - `Controllers/` - controller classes
  - `Repositories/` - repository classes
  - `SelectManagers/` - select manager classes
  - `Services/` - service classes
  - `Core/` - core classes and interfaces
  - `Entities/` - entity classes
  - `EntryPoints/` - entry point classes
  - `Hooks/` - hook classes
  - `Modules/` - core module files
  - `Resources/` - resource files (configs, layouts, templates, i18n)
  - `Tools/` - tool classes
- `client/` - frontend files
  - `lib/` - compiled dependencies
  - `modules/crm/` - crm module frontend files
  - `src/` - frontend source code
    - `views/` - view classes
    - `models/` - model classes
    - `controllers/` - controller classes

## Dependency Injection

- Use constructor injection.
- Type-hint dependencies in constructor parameters.
- Use interfaces for dependency declarations when possible.

### Examples

```php
use Espo\Core\Utils\Config;
use Espo\Core\Utils\Metadata;

class SomeClass
{
    private $config;
    private $metadata;
    
    public function __construct(Config $config, Metadata $metadata)
    {
        $this->config = $config;
        $this->metadata = $metadata;
    }
}
```

## Entities

- Extend `Espo\Core\Templates\Entities\Base`.
- Define fields in `Fields` class constant.
- Define relations in `Relations` class constant.
- Use getter methods for attribute access.

### Examples

```php
use Espo\Core\Templates\Entities\Base;

class Account extends Base
{
    const ENTITY_TYPE = 'Account';
    
    const FIELDS = [
        'id' => ['type' => Entity::ID],
        'name' => ['type' => Entity::VARCHAR, 'required' => true],
        'assignedUser' => ['type' => Entity::FOREIGN, 'relation' => 'assignedUser'], 
    ];
    
    const RELATIONS = [
        'assignedUser' => [
            'type' => Entity::BELONGS_TO,
            'entity' => 'User',
            'key' => 'assignedUserId',
        ],
        'meetings' => [
            'type' => Entity::HAS_MANY,
            'entity' => 'Meeting',
            'foreignKey' => 'parentId',
            'foreignType' => 'parentType',
        ],
    ];
    
    public function getName() : ?string
    {
        return $this->get('name');    
    }
}
```

## Repositories

- Extend `Espo\Core\Templates\Repositories\Base`
- Define methods for fetching records with specific conditions

### Examples

```php
use Espo\Core\Templates\Repositories\Base;
use Espo\ORM\Entity;

class AccountRepository extends Base
{    
    public function getByName(string $name) : ?Entity
    {
        return $this->where(['name' => $name])->findOne();
    }
}
```

## Controllers

- Extend `Espo\Core\Templates\Controllers\Base`
- Define action methods (must have "Action" suffix)
- Use `Espo\Core\Api\Request` and `Espo\Core\Api\Response`
- Throw exceptions for errors (e.g. `Espo\Core\Exceptions\BadRequest`, `Espo\Core\Exceptions\Forbidden`)

### Examples

```php
use Espo\Core\Api\Request;
use Espo\Core\Api\Response;
use Espo\Core\Controllers\Record;
use Espo\Core\Exceptions\Forbidden;

class AccountController extends Record
{
    public function getActionHelloWorld(Request $request) : Response
    {  
        if (!$this->getUser()->isAdmin()) {
            throw new Forbidden();
        }

        return $this->respondWithText('Hello World!');
    }
}
```

## Services

- Extend `Espo\Core\Templates\Services\Base`
- Inject dependencies
- Define public service methods
- Use hooks for pre and post processing

### Examples

```php
use Espo\Core\Exceptions\Forbidden;
use Espo\Core\Templates\Services\Base;
use Espo\ORM\Entity;

class AccountService extends Base
{
    protected function beforeCreateEntity(Entity $entity, $data)
    {        
        if (!$this->getUser()->isAdmin()) {
            throw new Forbidden();
        }
    }

    public function createSample(string $name) : Entity
    {
        $entity = $this->getEntityManager()->getEntity('Account');
        $entity->set('name', $name);
        $this->getEntityManager()->saveEntity($entity);

        return $entity;  
    }
}
```

## Hooks

- Implement `Espo\Core\Interfaces\Injectable`
- Use `static::` to access dependencies
- Define methods matching hook names (e.g. `afterSave`)

### Examples

```php
namespace Espo\Hooks\Note;

use Espo\ORM\Entity;

class TargetHandler extends \Espo\Core\Hooks\Base
{
    public function afterSave(Entity $entity, array $options)
    {        
        if (!empty($entity->get('parentId'))) {
            $target = $this->getEntityManager()->getEntity($entity->get('parentType'), $entity->get('parentId'));
            $target->set('hasNotes', true);
            $this->getEntityManager()->saveEntity($target);
        }
    }
}
```

## Frontend

- Define views in separate files in `client/src/views/` 
- Extend `Espo.View`
- Use `.js` extension
- Access models via `this.model`, `this.collection`
- Define events in `events` property

### Examples

```js
Espo.define('views/account/record/detail', ['views/record/detail'], function (Dep) {
    return Dep.extend({    
        events: {
            'click .action[data-action="convertToLead"]': function () {
                this.actionConvertToLead();
            },
        },

        actionConvertToLead: function () {            
            var model = this.model;
            
            Espo.Ajax.postRequest('Account/action/convertToLead', {
                id: model.id
            }).then(function (attributes) {
                var viewName = this.getMetadata().get('clientDefs.Lead.recordViews.detail') || 'views/lead/record/detail';
                
                this.getRouter().navigate('#Lead/view/' + attributes.id, {trigger: false});

                this.getHelper().createView(viewName, {
                    model: this.getModelFactory().create('Lead', attributes),
                    modelName: 'Lead',
                    id: attributes.id,
                    rootUrl: this.options.rootUrl,
                }, function (view) {
                    view.render();
                });
            }.bind(this));
        }
    });
});
```

## Metadata

- Stored in PHP files in `application/Espo/Resources/metadata/`
- Defined as nested associative arrays
- Accessible in backend via `$this->getMetadata()->get()`
- Accessible in frontend via `this.getMetadata().get()`

### Examples

```php
// application/Espo/Resources/metadata/entityDefs/Account.json
{
    "fields": {        
        "name": {
            "type": "varchar",
            "maxLength": 150,
            "required": true
        },
        "website": {
            "type": "url"
        },
        "emailAddress": {
            "type": "email"
        }
    },
    "links": {        
        "meetings": {
            "type": "hasMany",
            "entity": "Meeting",
            "foreign": "account"
        }
    }
}
```

```js
// client/src/views/account/record/detail.js
    setup: function () {
        var meetingsField = this.getMetadata().get('entityDefs.Account.fields.meetings');
        if (meetingsField && meetingsField.disabled) {
            this.hideField('meetings');
        }        
    },
```

## Layouts

- Stored in JSON files in `application/Espo/Modules/<ModuleName>/Resources/layouts/`
- Defined as nested arrays
- Used to define field positioning, panel arrangement, etc.

### Examples

```json
// application/Espo/Modules/Crm/Resources/layouts/Account/detail.json
[
    {
        "label": "",
        "rows": [
            [
                {"name": "name"},
                {"name": "website"}
            ],
            [
                {"name": "emailAddress"},
                false
            ]
        ]
    }
]
```

## Translations

- Stored in JSON files in `application/Espo/Resources/i18n/<language>/`
- Defined as nested objects
- Accessible in backend via `$this->getLanguage()->translate()`
- Accessible in frontend via `this.translate()`

### Examples

```json
// application/Espo/Resources/i18n/en_US/Account.json 
{
    "fields": {
        "name": "Name",
        "website": "Website",
        "emailAddress": "Email"
    },
    "links": {
        "meetings": "Meetings"
    }
}
```

```php
// In a backend controller

$this->getLanguage()->translate('Account', 'scopeNames');
```

```js
// In a frontend view
this.translate('Create Account', 'labels', 'Account');
```

## Backend Utils

### Config

- Allows access to system configuration
- Accessible via `$this->getConfig()` in backend classes
- Get value: `$value = $this->getConfig()->get('paramName')`
- Has value: `if ($this->getConfig()->has('paramName')) { ... }`

### Metadata

- Allows access to metadata
- Accessible via `$this->getMetadata()` in backend classes  
- Get value: `$fieldDefs = $this->getMetadata()->get(['entityDefs', 'Account', 'fields', 'name'])`

### Language

- Allows translating labels
- Accessible via `$this->getLanguage()` in backend classes
- Basic usage: `$translated = $this->getLanguage()->translate('originalLabel', 'category')`

### AclManager

- Allows checking user access
- Accessible via `$this->getAclManager()` in backend classes
- Check: `$hasAccess = $this->getAclManager()->check($user, $scope, $action)`

### SelectBuilderFactory

- Allows creating a select query builder
- Accessible via `$this->getSelectBuilderFactory()` in backend classes
- Create: `$selectBuilder = $this->getSelectBuilderFactory()->create()`

### EntityManager

- Allows fetching and storing entities
- Accessible via `$this->getEntityManager()` in backend classes
- Get entity: `$entity = $this->getEntityManager()->getEntity($entityType, $id)`
- Get Repository: `$repository = $this->getEntityManager()->getRepository($entityType)`
- Save entity: `$this->getEntityManager()->saveEntity($entity)`

## Frontend Helpers

### ViewHelper

- Provides utility methods for views 
- Accessible via `this.getHelper()` in frontend views

#### Examples:

- Create view: `this.getHelper().createView(viewName, options, callback)`
- Datetime utils: `this.getHelper().date`
- Number utils: `this.getHelper().number`

### LanguageHelper 

- Provides translation methods
- Accessible via `this.getLanguage()` in frontend views

#### Examples:

- Translate: `this.getLanguage().translate(label, category, scope)`

### ModelFactory

- Allows creating models
- Accessible via `this.getModelFactory()` in frontend views

#### Examples:

- Create model: `this.getModelFactory().create(modelName, callback)`

### CollectionFactory

- Allows creating collections
- Accessible via `this.getCollectionFactory()` in frontend views

#### Examples:

- Create collection: `this.getCollectionFactory().create(collectionName, callback)`

### FieldManager

- Provides utility methods for fields
- Accessible via `this.getFieldManager()` in frontend views

#### Examples:

- Get field view name: `this.getFieldManager().getViewName(type)`

### Router

- Provides routing functionality
- Accessible via `this.getRouter()` in frontend views

#### Examples:

- Navigate: `this.getRouter().navigate(url, options)`
- Get current URL: `this.getRouter().getCurrentUrl()`

## Select Builder

- Used for building select queries
- Methods can be chained

### Examples

```php
$query = $this->getSelectBuilderFactory()
    ->create()
    ->from('Account')
    ->withStrictAccessControl()
    ->buildQueryBuilder()
    ->select(['id', 'name'])
    ->where(['type' => 'Customer'])
    ->order('name')
    ->build();

$pdo = $this->getEntityManager()->getPDO();
$sql = $query->getSql();
$sth = $pdo->prepare($sql);
$sth->execute();
$rows = $sth->fetchAll(PDO::FETCH_ASSOC);
```

## ORM

- Object-relational mapping
- Allows fetching and storing entities

### Examples

```php
$entityManager = $this->getEntityManager();
$repository = $entityManager->getRepository('Account');

$entity = $repository->get($id);
$entity->set('name', 'New Name');
$entityManager->saveEntity($entity);

$accountList = $repository
    ->where(['type' => 'Customer'])
    ->limit(0, 10)
    ->order('name', 'ASC')
    ->find();

$accountList = $repository
    ->distinct()
    ->join('teams')
    ->where([
        'Teams.id' => $teamId,
        'name*' => '%' . $query . '%',        
    ])
    ->find();
```

## Testing

- Backend tests stored in `tests/unit/Espo/`
- Extend `\PHPUnit\Framework\TestCase`

### Examples

```php
namespace tests\unit\Espo\Core;

use Espo\Core\Utils\Config;

class ConfigTest extends \PHPUnit\Framework\TestCase
{
    protected $object;

    protected $objects;

    protected function setUp() : void
    {
        $this->objects['fileManager'] = $this->getMockBuilder('\Espo\Core\Utils\File\Manager')->disableOriginalConstructor()->getMock();

        $this->object = new Config($this->objects['fileManager']);
    }

    public function testGet()
    {
        $this->object->set('test', 'value');
        $this->assertEquals('value', $this->object->get('test'));

        $this->object->set('testObject', (object) ['test' => 'value']);
        $this->assertEquals('value', $this->object->get('testObject.test'));
    }
}
```

This documentation covers key aspects of EspoCRM development, including:

- Class naming conventions
- Directory structure 
- Dependency injection
- Entities
- Repositories
- Controllers
- Services 
- Hooks
- Frontend views
- Metadata
- Layouts
- Translations
- Backend utilities (Config, Metadata, Language, AclManager, SelectBuilderFactory, EntityManager) 
- Frontend helpers (ViewHelper, LanguageHelper, ModelFactory, CollectionFactory, FieldManager, Router)
- Select Builder for query building
- ORM for fetching and storing entities
- Testing with

## Advanced Workflows

- Defined in `application/Espo/Modules/Advanced/Core/Workflow/`
- Consist of conditions, actions, and formulas
- Used for automating business processes

### Examples

```php
// application/Espo/Modules/Advanced/Core/Workflow/Actions/CreateEntity.php

namespace Espo\Modules\Advanced\Core\Workflow\Actions;

use Espo\Core\Exceptions\Error;
use Espo\Core\Workflow\Actions\Base;
use Espo\Core\Workflow\Utils;
use Espo\ORM\Entity;

class CreateEntity extends Base
{
    protected function run(Entity $entity, $actionData)
    {
        $entityManager = $this->getEntityManager();

        $entityType = $this->evaluate($actionData->entityType, $entity);
        $value = $this->evaluate($actionData->value, $entity);

        if (empty($entityType)) {
            throw new Error("Empty target entity type.");
        }

        $target = $entityManager->getEntity($entityType);

        if (isset($value)) {
            $target->set($value);
        }

        if (!empty($actionData->formula)) {
            $scriptData = $this->getFormulaManager()->run($target, $actionData->formula, $this->getFormulaVariables());
            Utils::fillEntityFromArray($target, $scriptData);
        }

        if (isset($actionData->link) && isset($actionData->linkField)) {
            $link = $actionData->link;
            $field = $actionData->linkField;
            $target->set($field . 'Id', $entity->id);
            $entityManager->saveEntity($target);
            $entity->setFetched($link, $target);
        } else {
            $entityManager->saveEntity($target);
        }
    }
}
```

## Formula Functions

- Defined in `application/Espo/Core/Formula/Functions/`
- Used in formula fields, workflow conditions and actions, etc.
- Accessible in formulas via function name (e.g. `IF(field, 1, 0)`)

### Examples

```php
// application/Espo/Core/Formula/Functions/IF.php

namespace Espo\Core\Formula\Functions\Logical;

use Espo\Core\Formula\{
    Functions\BaseFunction,
    ArgumentList,
};

class IF extends BaseFunction
{
    public function process(ArgumentList $args)
    {
        if (count($args) < 2) {
            $this->throwTooFewArguments(2);
        }

        if (count($args) > 3) {
            $this->throwTooManyArguments(3);
        }

        $condition = $this->evaluate($args[0]);

        if ($condition) {
            return $this->evaluate($args[1]);
        }

        if (count($args) === 2) {
            return null;
        }

        return $this->evaluate($args[2]);
    }
}
```

## Scheduled Jobs

- Defined in `application/Espo/Jobs/`
- Used for running background tasks on schedule
- Can be managed in admin panel

### Examples

```php
// application/Espo/Jobs/Cleanup.php

namespace Espo\Jobs;

use Espo\Core\Jobs\Base;

class Cleanup extends Base
{
    public function run()
    {
        $period = '-' . $this->getConfig()->get('cleanupPeriod', '1 month');
        $datetime = new \DateTime();
        $datetime->modify($period);

        $this->cleanupEntity('Notification', $datetime);
        $this->cleanupEntity('ScheduledJob', $datetime, function ($builder) {
            $builder->where('status', '=', 'Success');
        });
        $this->cleanupEntity('ScheduledJobLogRecord', $datetime);
    }

    protected function cleanupEntity($entityType, $datetime, $additionalBuilder = null)
    {
        $repository = $this->getEntityManager()->getRepository($entityType);
        $builder = $repository->where([
            'createdAt<' => $datetime->format('Y-m-d H:i:s'),
        ]);

        if (!empty($additionalBuilder)) {
            $additionalBuilder($builder);
        }

        $ids = $builder->limit(0, 1000)->find()->toArray();

        if (!empty($ids)) {
            $repository->remove($ids);
        }
    }
}
```

## Migrations

- Stored in `application/Espo/Core/Utils/Database/Schema/`
- Used for making changes to database schema
- Can be executed via CLI command `php command.php database-update`

### Examples

```php
// application/Espo/Core/Utils/Database/Schema/rebuildPath.php

$pdo = $this->getEntityManager()->getPDO();
$query = "
    ALTER TABLE `email` 
    ADD `parent_id` VARCHAR(24) DEFAULT NULL COLLATE utf8mb4_unicode_ci,
    ADD `parent_type` VARCHAR(100) DEFAULT NULL COLLATE utf8mb4_unicode_ci
";
$pdo->exec($query);

```

## CLI Commands

- Defined in `application/Espo/Commands/`
- Can be executed via `php command.php <command>`
- Used for running tasks from command line

### Examples

```php
// application/Espo/Commands/Import.php

namespace Espo\Commands;

use Espo\Core\{
    Exceptions\Error,
    Console\Command,
    Console\Params,
    Console\IO,
};

use Espo\Core\Utils\File\Manager as FileManager;

class Import implements Command
{
    protected $fileManager;

    public function __construct(FileManager $fileManager)
    {
        $this->fileManager = $fileManager;
    }

    public function run(Params $params, IO $io) : void
    {
        $filePath = $params->getOption('file');
        $delimiter = $params->getOption('delimiter', ',');
        $enclosure = $params->getOption('enclosure', '"');
        $entityType = $params->getOption('entity-type');

        if (!$filePath || !$entityType) {
            throw new Error("Not enough params.");
        }

        $fileContents = $this->fileManager->getContents($filePath);

        $importParams = [
            'entityType' => $entityType,
            'fileContents' => $fileContents,
            'delimiter' => $delimiter,
            'enclosure' => $enclosure,
        ];

        $import = $this->getServiceFactory()->create('Import')->importFeed($importParams);

        $io->writeLine("Imported: " . $import->countCreated);
        $io->writeLine("Updated: " . $import->countUpdated);
        $io->writeLine("Duplicates: " . $import->countDuplicates);
    }
}
```

## PDF Templates

- Stored in `application/Espo/Resources/templates/`
- Used for generating PDF documents
- Can be managed in admin panel

### Examples

```html
<!-- application/Espo/Resources/templates/quote/default.tpl -->

<!doctype html>
<html>
<head>
<title>{{quote.id}} - {{quote.name}}</title>
<style type="text/css">
body {
    font-family: 'DejaVu Sans', sans-serif;
}
</style>
</head>
<body>
<div>
<div><h1>{{quote.name}}</h1></div>
<div>
<p>Amount: {{quote.amount}}</p>
<p>Valid Until: {{quote.validUntil}}</p>
</div>
</div>
</body>
</html>
```

## Prompt Usage Suggestions

Here are some ways you could use this documentation to assist with prompts:

1. Provide code snippets to demonstrate how to implement specific functionality, such as creating a custom API endpoint, defining a new entity, or adding a hook.

Example:
```
User: How can I create a custom API endpoint in EspoCRM?
