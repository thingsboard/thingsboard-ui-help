#### Custom action function

<div class="divider"></div>
<br/>

*function ($event, widgetContext, entityId, entityName, additionalParams, entityLabel): void*

A JavaScript function performing custom action.

**Parameters:**

<ul>
  {% include widget/action/custom_action_args %}
</ul>

<div class="divider"></div>

##### Examples

* Display alert dialog with entity information:

```javascript
{:code-style="max-height: 300px;"}
var title;
var content;
if (entityName) {
    title = entityName + ' details';
    content = '<b>Entity name</b>: ' + entityName;
    if (additionalParams && additionalParams.entity) {
        var entity = additionalParams.entity;
        if (entity.id) {
            content += '<br><b>Entity type</b>: ' + entity.id.entityType;
        }
        if (!isNaN(entity.temperature) && entity.temperature !== '') {
            content += '<br><b>Temperature</b>: ' + entity.temperature + ' °C';
        }
    }
} else {
    title = 'No entity information available';
    content = '<b>No entity information available</b>';
}

showAlertDialog(title, content);
 
function showAlertDialog(title, content) {
    setTimeout(function() {
      widgetContext.dialogs.alert(title, content).subscribe();
    }, 100);
}
{:copy-code}
```

* Delete device after confirmation:

```javascript
{:code-style="max-height: 300px;"}
var $injector = widgetContext.$scope.$injector;
var dialogs = $injector.get(widgetContext.servicesMap.get('dialogs'));
var deviceService = $injector.get(widgetContext.servicesMap.get('deviceService'));

openDeleteDeviceDialog();

function openDeleteDeviceDialog() {
  var title = 'Are you sure you want to delete the device ' + entityName + '?';
  var content = 'Be careful, after the confirmation, the device and all related data will become unrecoverable!';
  dialogs.confirm(title, content, 'Cancel', 'Delete').subscribe(
    function(result) {
      if (result) {
        deleteDevice();
      }
    }
  );
}

function deleteDevice() {
  deviceService.deleteDevice(entityId.id).subscribe(
    function() {
      widgetContext.updateAliases();
    }
  );
}
{:copy-code}
```

* Open state conditionally with saving particular state parameters

```javascript
{:code-style="max-height: 300px;"}
var entitySubType;
var $injector = widgetContext.$scope.$injector;
$injector.get(widgetContext.servicesMap.get('entityService')).getEntity(entityId.entityType, entityId.id)
  .subscribe(function(data) {
    entitySubType = data.type;
    if (entitySubType == 'energy meter') {
      openDashboardStates('energy_meter_details_view');
    } else if (entitySubType == 'thermometer') {
      openDashboardStates('thermometer_details_view');
    }
});

function openDashboardStates(statedId) {
  var stateParams = widgetContext.stateController.getStateParams();
  var params = {
    entityId: entityId,
    entityName: entityName
  };

  if (stateParams.city) {
    params.city = stateParams.city;
  }

  widgetContext.stateController.openState(statedId, params, false);
}
{:copy-code}
```

* Go back to the first state, after this go to the target state

```javascript
{:code-style="max-height: 300px;"}
var stateIndex = widgetContext.stateController.getStateIndex();
while (stateIndex > 0) {
  stateIndex -= 1;
  backToPrevState(stateIndex);
}
openDashboardState('devices');

function backToPrevState(stateIndex) {
  widgetContext.stateController.navigatePrevState(stateIndex);
}

function openDashboardState(statedId) {
  var currentState = widgetContext.stateController.getStateId();
  if (currentState !== statedId) {
    var params = {};
    widgetContext.stateController.updateState(statedId, params, false);
  }
}
{:copy-code}
```

* Copy device access token to buffer

```javascript
{:code-style="max-height: 300px;"}
var $injector = widgetContext.$scope.$injector;
var deviceService = $injector.get(widgetContext.servicesMap.get('deviceService'));
var $translate = $injector.get(widgetContext.servicesMap.get('translate'));
var $scope = widgetContext.$scope;
if (entityId.id && entityId.entityType === 'DEVICE') {
  deviceService.getDeviceCredentials(entityId.id, true).subscribe(
    (deviceCredentials) => {
      var credentialsId = deviceCredentials.credentialsId;
      if (copyToClipboard(credentialsId)) {
        $scope.showSuccessToast($translate.instant('device.accessTokenCopiedMessage'), 750, "top", "left");
      }
    }
  );
}

function copyToClipboard(text) {
  if (window.clipboardData && window.clipboardData.setData) {
    return window.clipboardData.setData("Text", text);
  }
  else if (document.queryCommandSupported && document.queryCommandSupported("copy")) {
    var textarea = document.createElement("textarea");
    textarea.textContent = text;
    textarea.style.position = "fixed";
    document.body.appendChild(textarea);
    textarea.select();
    try {
      return document.execCommand("copy");
    }
    catch (ex) {
      console.warn("Copy to clipboard failed.", ex);
      return false;
    }
    document.body.removeChild(textarea);
  }
}
{:copy-code}
```
