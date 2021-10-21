# Example of use case scenarios for Qliksense Javascript API

**By Carlo**

# Intro

I'm writing this document because i've found the qliksense documentation difficult to decipher and even more difficult to get started. The documentation is also difficult to navigate and the library is full of quirks that a lot of modern libraries just don't have.

Hopefully these examples help you with your qliksense js woes.

## Must do

The below html is required to access the connection script. You'll have something similar and not exactly like this, so get it from your application, because it will be different.

```html
<link
  rel="stylesheet"
  href="https://your-tenant.us.qlikcloud.com/resources/autogenerated/qlik-styles.css"
/>
<script src="https://your-tenant.us.qlikcloud.com/resources/assets/external/requirejs/require.js"></script>
```

# App connection

Here is, by default, an example of what you need to connect to a qliksense app.

```javascript
var config = {
  host: 'myhost.com',
  prefix: '/',
  port: window.location.port,
  isSecure: true,
};

require.config({
  baseUrl:
    (config.isSecure ? 'https://' : 'http://') +
    config.host +
    (config.port ? ':' + config.port : '') +
    config.prefix +
    'resources',
});

require(['js/qlik'], function (qlik) {
  qlik.setOnError(function (error) {
    console.log(error);
  });

  var app = qlik.openApp('MasteringQlikSense.qvf', config);
});
```

Here's a slight modification to make it behave better for modern JS applications

```javascript
let config = {
  host: 'myhost.com',
  prefix: '/',
  port: window.location.port,
  isSecure: true,
};

export function qliksenseAppConnection(appId = '') {
  return new Promise((resolve, reject) => {
    window.require.config({
      baseUrl:
        (config.isSecure ? 'https://' : 'http://') +
        config.host +
        (config.port ? ':' + config.port : '') +
        config.prefix +
        'resources',
    });

    window.require(['js/qlik'], function (qlik) {
      qlik.setOnError(function (error) {
        reject(error);
        console.log(error);
      });

      resolve(qlik.openApp(appId, config));
    });
  });
}
```

With these modifications, you can connect to multiple qliksense apps which returns a promise that you can `await` or `.then`

Using `window.require` allows you to access the requireJS that is included on the page template from within modern js apps that utilises React, Angular, etc.

**React Example:**

```javascript
import { useState, useEffect } from 'react';
import { qliksenseAppConnection } from '../util/qliksenseUtil.js';


export default ChartExample(props) {
  const [app, setApp] = useState(false);

  useEffect(() => {
    setApp(qliksenseAppConnection('appIdGoesHere'));
  }, [])

  ...

}
```

# Display a chart/object

The easiest way to do this is via iframes. Then you won't need anything else, just the iframe. But if you don't want that, here's what you do.

**React Example:**

```javascript
import { useState, useEffect } from 'react';
import { qliksenseAppConnection } from '../util/qliksenseUtil.js';


export default ChartExample(props) {
  const [app, setApp] = useState(false);

  useEffect(() => {
    setApp(qliksenseAppConnection('appIdGoesHere'));
  }, [])


  render() {
    if(app) {
      app.getObject('divIdGoesHere', 'objectIdGoesHere', {
        noInteration: false,
        noSelections: true
      })
    }

    return (
      <div id="divIdGoesHere">Loading...</div>
    )
  }
}
```

`divIdGoesHere` can be replaced with a selector, eg `document.getElementById('divIdGoesHere')`. It's easier just to use the objectID as the div id. Although this obviously wouldn't work if you wanted the same chart multiple times on the page (???).

# Programmatically apply a filter to an app

So filters only work across the entire app, and I don't think you can apply a filter to just one object within the app.

**React Example:**

```javascript
import { useState, useEffect } from 'react';
import { qliksenseAppConnection } from '../util/qliksenseUtil.js';


export default ChartExample(props) {
  const [app, setApp] = useState(false);

  useEffect(() => {
    setApp(qliksenseAppConnection('appIdGoesHere'));
  }, [])

  setCountry(country = '') {
    if(app) {
      app.field('FieldNameGoesHere').clear().then(() => {
        app.field('FieldNameGoesHere').selectValues([country], true, true)
      })
    }
  }

  render() {

    setCountry('Australia');
  ...

}
```

By default in qliksense, filters are additive. So if you call `.selectValues` multiple times with different values, it's like adding a `OR` clause to that field, eg Australia OR Spain. We get around this, but clearing the field of all filters, THEN applying our own.

# Get the data used to create the object/chart

Because you can't always get the charts looking the way you want, or you want to create a suplemental chart, here is a way to retrieve the data used to create a table/chart.

https://community.qlik.com/t5/Integration-Extension-APIs/Get-data-from-table-or-chart/td-p/1241605

```javascript
import { useState, useEffect } from 'react';
import { qliksenseAppConnection } from '../util/qliksenseUtil.js';
import OtherChartComponent from '../components/OtherChartComponent';


export default ChartExample(props) {
  const [app, setApp] = useState(false);
  const [chartData, setChartData] = useState([]);

  useEffect(() => {
    setApp(qliksenseAppConnection('appIdGoesHere'));
  }, [])

  getData(objectId) {
    app.getObject(objectId).then(model => {
      console.log(model.layout.qHyperCube.qSize); // Details of the pagination for the object

      // I don't know what these params do
      model.getHyperCubeData('/qHyperCubeDef', [{
         qTop: 0,
         qLeft: 0,
         qWidth: 10,
         qHeight: 10000
       }]).then(objectData => {
         console.log(objectData);
         setChartData(objectData);
       })
    })
  }

  render() {
    if(app) {
      getData('ObjectIdGoesHere');
    }

    return (<OtherChartComponent data={chartData} />)
  }
}

```

I don't know why it's this complicated to retrieve the objects data. It also gives you the bear minimum of information to get the chart working and not the full data set.

# Connecting to multiple apps

```javascript
import { useState, useEffect } from 'react';
import { qliksenseAppConnection } from '../util/qliksenseUtil.js';


export default ChartExample(props) {
  const [app, setApp] = useState(false);
  const [app2, setApp2] = useState(false);

  useEffect(() => {
    setApp(qliksenseAppConnection('appIdGoesHere'));
    setApp2(qliksenseAppConnection('app2IdGoesHere'));
  }, [])

  render() {
    if(app) {
      app.getObject('divIdGoesHere', 'appObjectIdGoesHere');
      app2.getObject('div2IdGoesHere', 'app2ObjectIdGoesHere')
    }
    
    ...
    
  }
}
```
