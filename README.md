### Usando IONIC com TypeOrm - Portugues(pt-br) / Using ionic with typeorm - Portuguese

referencias
https://github.com/typeorm/ionic-example

ESTE TUTORIAL É VOLTADO PARA O IONIC VERSAO 3

Criando projeto ionic3 = ionic start {NOME_DO_PROJETO} blank --type=ionic-angular

1. instale o plugin
```bash
ionic cordova plugin add cordova-sqlite-storage --save
```

2. inslate o typeorm
```bash
npm install typeorm --save
```

3. instale node.js-Types
```bash
npm install @types/node --save-dev
```

4. adicione  `"typeRoots": ["node_modules/@types"]` em `tsconfig.json` na chave `"compilerOptions"`

5. crie `webpack.config.js`, de preferencia dentro da uma pasta "config" no root do projeto
>>>Adiciona o conteudo abaixo dentro de `webpack.config.js`
```bash
var webpack = require('webpack');
var ionicWebpackFactory = require(process.env.IONIC_WEBPACK_FACTORY);

var ModuleConcatPlugin = require('webpack/lib/optimize/ModuleConcatenationPlugin');
var PurifyPlugin = require('@angular-devkit/build-optimizer').PurifyPlugin;

var useDefaultConfig = require('@ionic/app-scripts/config/webpack.config.js');

useDefaultConfig.dev.plugins = [
  ionicWebpackFactory.getIonicEnvironmentPlugin(),
  ionicWebpackFactory.getCommonChunksPlugin(),
  new webpack.NormalModuleReplacementPlugin(/typeorm$/, function (result) {
    result.request = result.request.replace(/typeorm/, "typeorm/browser");
  }),
  new webpack.ProvidePlugin({
    'window.SQL': 'sql.js/js/sql.js'
  })
]

useDefaultConfig.prod.plugins = [
  ionicWebpackFactory.getIonicEnvironmentPlugin(),
  ionicWebpackFactory.getCommonChunksPlugin(),
  new ModuleConcatPlugin(),
  new PurifyPlugin(),
  new webpack.NormalModuleReplacementPlugin(/typeorm$/, function (result) {
    result.request = result.request.replace(/typeorm/, "typeorm/browser");
  })
]

module.exports = function () {
  return useDefaultConfig;
};
```

6. adicione a chave `"config"` no `package.json` apontando para o `webpack.config.js` criado acima
```bash
"config": {
    "ionic_webpack": "./config/webpack.config.js"
  },
```
TypeORM implementado no projeto !

### EXEMPLO DE PRIMEIRAS CONFIGURAÇÕES INICIAIS E RECOMENDAÇÕES/ORGANIZAÇÃO

Crie uma pasta dentro de src chamada entities, dentro coloque as entidades
Exemplo: src/entities/category.ts

```bash
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity('category')
export class Category {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

}
```

Faça a conexao dentro de `src/app/app.component.ts`
Exemplo:
*src/app/app.component.ts
```bash
import { Component } from '@angular/core';
import { Platform } from 'ionic-angular';
import { StatusBar } from '@ionic-native/status-bar';
import { SplashScreen } from '@ionic-native/splash-screen';
import { createConnection } from 'typeorm'

import { HomePage } from '../pages/home/home';

import { Author } from '../entities/author';
import { Category } from '../entities/category';
import { Post } from '../entities/post';

@Component({
  templateUrl: 'app.html'
})
export class MyApp {
  rootPage: any;

  constructor(platform: Platform, statusBar: StatusBar, splashScreen: SplashScreen) {
    platform.ready().then(async () => {
      // Okay, so the platform is ready and our plugins are available.
      // Here you can do any higher level native things you might need.
      statusBar.styleDefault();
      splashScreen.hide();

      // Depending on the machine the app is running on, configure
      // different database connections
      if(platform.is('cordova')) {
        // Running on device or emulator
        await createConnection({
          type: 'cordova',
          database: 'test',
          location: 'default',
          logging: ['error', 'query', 'schema'],
          synchronize: true,
          entities: [
            Author,
            Category,
            Post
          ]
        });
      } else {
        // Running app in browser
        await createConnection({
          type: 'sqljs',
          autoSave: true,
          location: 'browser',
          logging: ['error', 'query', 'schema'],
          synchronize: true,
          entities: [
            Author,
            Category,
            Post
          ]
        });
      }

      splashScreen.hide();
      this.rootPage = HomePage;
    });
  }
}
```

ATENTE-SE PARA `rootPage:any = null;` sem nenhuma "view" associada ao inicio da classe(app.component.ts), isso serve para primeiramente ser feito a conexao com o banco para somente depois a view ser iniciada

7 instale ' "sql.js": "^0.5.0" ' no package.json em devDependencies...
Exemplo:
"devDependencies": {
	...
    "sql.js": "^0.5.0",
	...
  },
  
8 Se o projeto estiver em execução (ionic serve) , faça o re-build , finalizando a execução do projeto e re-executando novamente

>> é possivel ver a execução dos processos no console do navegador
obs : nunca testei este projeto em um dispositivo movel

FIM !!

### Atenção !!
Em ambiente de PRODUÇÃO é recomendado usar a chave `synchronize` como `false`, veja o exemplo de conexao abaixo:
```bash
await createConnection({
          type: 'cordova',
          database: 'mylistdatabase',
          location: 'default',
          logging: ['error', 'query', 'schema'],
          synchronize: false,
          entities: [
            Category
          ]
          migrations: [
            SUA_MIGRATION_AQUI,
            SUA_MIGRATION_AQUI2,
          ],
          migrationsRun: true
```

ERROS_BUGANTES....
{
Typescript Error
';' expected.
undefined
}

Caso o erro acima acontecer, entre em "app.component.ts", dê um crlt+s , pronto !


CORRIGINDO O ERRO ACIMA.....
Em seu "package.json" em, procure onde esta a chave "typescript", use a versao: "^3.1.1"
Exemplo: 
"devDependencies": {
	...
    "typescript": "^3.1.1"
  },
 
delete a node_modules e package-lock.json, feche o projeto se ele estiver aberto em algum editor de codigo, abra o terminal e use
npm install no projeto novamente

ERROS_BUGANTES 2....
Se der algum erro estranho, "use versoes fixas" no `package.json`
Atenção nas versões do typeorm | typescript | @ionic/app-scripts
exemplo:
`"typeorm": "0.2.17"` OU `"typeorm": "0.2.31"`
`"@ionic/app-scripts": "3.2.1"`
`"typescript": "3.4.5",`


INFORMAÇÕES EXTRAS
### Limitations to TypeORM when using production builds

Since Ionic make a lot of optimizations while building for production, the following limitations will occur:

1. Entities have to be marked with the table name (eg `@Entity('table_name')`)

2. `getRepository()` has to be called with the name of the entity instead of the class (*eg `getRepository('post') as Repository<Post>`*)





