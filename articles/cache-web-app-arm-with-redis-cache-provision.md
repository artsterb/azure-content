<properties 
	pageTitle="Provision Web App with Redis Cache" 
	description="Use Azure Resource Manager template to deploy web app with Redis Cache." 
	services="redis-cache" 
	documentationCenter="" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="cache" 
	ms.workload="web" 
	ms.tgt_pltfrm="cache-redis" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/02/2015" 
	ms.author="tomfitz"/>

# Create a Web App plus Redis Cache using a template

In this topic, you will learn how to create an Azure Resource Manager template that deploys an Azure Web App with Redis cache. You will learn how to define which resources are deployed and 
how to define parameters that are specified when the deployment is executed. You can use this template for your own deployments, or customize it to meet your requirements.

For more information about creating templates, see [Authoring Azure Resource Manager Templates](resource-group-authoring-templates.md).

For the complete template, see [Web App with Redis Cache template](https://github.com/tfitzmac/AppServiceTemplates/blob/master/RedisCache_WebApp.json).

## What you will deploy

In this template, you will deploy:

- Azure Web App
- Azure Redis Cache.

## Parameters to specify

[AZURE.INCLUDE [app-service-web-deploy-web-parameters](../includes/app-service-web-deploy-web-parameters.md)]

[AZURE.INCLUDE [cache-deploy-parameters](../includes/cache-deploy-parameters.md)]



## Resources to deploy

[AZURE.INCLUDE [app-service-web-deploy-web-host](../includes/app-service-web-deploy-web-host.md)]

### Redis Cache

Creates the Azure Redics Cache that is used with the web app. The name of the cache is specified in the **redisCacheName** parameter.

The template creates the cache in the same location as the web app, which is recommended for best performance. 

    {
      "apiVersion": "2014-04-01-preview",
      "name": "[parameters('redisCacheName')]",
      "type": "Microsoft.Cache/Redis",
      "location": "[parameters('siteLocation')]",
      "properties": {
        "sku": {
          "name": "[parameters('redisCacheSKU')]",
          "family": "[parameters('redisCacheFamily')]",
          "capacity": "[parameters('redisCacheCapacity')]"
        },
        "redisVersion": "[parameters('redisCacheVersion')]",
        "enableNonSslPort": true
      }
    }

### Web app

Creates the web app with name specified in the **siteName** parameter.

Notice that the web app is configured with app setting properties that enable it to work with the Redis Cache. This app settings are dynamically created based on values provided during deployment.
        
    {
      "apiVersion": "2014-06-01",
      "name": "[parameters('siteName')]",
      "type": "Microsoft.Web/sites",
      "location": "[parameters('siteLocation')]",
      "dependsOn": [
        "[concat('Microsoft.Web/serverFarms/', parameters('hostingPlanName'))]"
      ],
      "properties": {
        "name": "[parameters('siteName')]",
        "serverFarm": "[parameters('hostingPlanName')]"
      },
      "resources": [
        {
          "apiVersion": "2014-04-01",
          "type": "config",
          "name": "web",
          "dependsOn": [
            "[concat('Microsoft.Web/sites/', parameters('siteName'))]"
          ],
          "properties": {
            "appSettings": [
              {
                "name": "REDIS_HOST",
                "value": "[concat(parameters('siteName'), '.redis.cache.windows.net:6379')]"
              },
              {
                "name": "REDIS_KEY",
                "value": "[listKeys(resourceId('Microsoft.Cache/Redis', parameters('siteName')), '2014-04-01').primaryKey]"
              }
            ]
          }
        }
      ]
    }     



## Commands to run deployment

[AZURE.INCLUDE [app-service-deploy-commands](../includes/app-service-deploy-commands.md)]

### PowerShell

    New-AzureResourceGroupDeployment -TemplateUri https://raw.githubusercontent.com/tfitzmac/AppServiceTemplates/master/RedisCache_WebApp.json -ResourceGroupName ExampleDeployGroup

### Azure CLI

    azure group deployment create --template-uri https://raw.githubusercontent.com/tfitzmac/AppServiceTemplates/master/RedisCache_WebApp.json -g ExampleDeployGroup


