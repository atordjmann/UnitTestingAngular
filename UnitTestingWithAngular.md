# UNIT TESTING IN ANGULAR

*Source principale : cours pluralsight “Unit testing in Angular”.*

# Introduction

L’objectif de ce document est de s’initier aux tests unitaires de composants Angular avec le framework de test Jest.
Un projet Angular possède initialement toute la configuration requise afin de tester les composants avec Jasmine et Karma. Il est ainsi très rapide et simple de s’essayer aux tests avec une application Angular. Il suffit d’écrire des tests dans les fichiers finissant par .spec.ts, de lancer la commande npm test, et le tour est joué.
Cependant nous avons choisi de nous intéresser au framework de test Jest, qui est un framework qui connait de plus en plus de succès de par sa simplicité, rapidité et efficacité. Même si ce framework a été créé par les développeurs Facebook pour tester les projets React, on peut l’utiliser dans des projets Typescript, Node, Babel, React, Angular, Vue…
Il existe de nombreuses raisons pour utiliser Jest comme framework de test. Il n’y a pas de configuration particulière pour utiliser Jest. Une fois installé, Jest peut être utilisé directement dans un projet. Les tests peuvent garder facilement la trace des gros objets, et on peut mocker les objets très facilement. Pour plus de rapidité, les tests sont mis en parallèle et exécutés dans des processus distincts afin d’augmenter les performances. Puis, en cas d’échec de test, Jest fourni un contexte riche.


# Généralités
## Architecture des tests
Afin de structurer les tests pour en faciliter la lecture, et repérer au mieux les tests qui ne passent pas, les erreurs... On utilise les mots-clés "describe" et "it".
- **"describe"** permet de définir une section de tests, qui se rapportent à un sujet commun.
- **"it"** permet de définir un test.

Pour structurer au mieux les tests, il est important de partir d'une phrase résumant le test. Par exemple: "user Service getUser method should retrieve the correct user". Cette phrase est ensuite transformée en:
```javascript
describe (‘user service’, () => {
          describe( ‘getUser method’,  ()   =>  {
                     it(‘should retrieve the correct user’)…
```

Les tests sont divisés en trois parties :
1)	Préconditions et inputs
2)	Agir sur les objets ou classes à tester
3)	Assertion du résultat

## Lancer les tests
`npm test`

## Code Coverage
Il peut être très intéressant de savoir si nos tests couvrent bien la totalité des composants. On peut parfois passer à côté d'une condition, ou d'une fonction. Puis, le rapport de couverture peut être la preuve d'un travail de test efficace, peut rassurer un client sur la qualité, et encourager à poursuivre les tests. Les framework de tests fournissent un moyen simple de génerer des rapports de couverture. Pour chaque composant testé, on peut voir sa couverture (avec le nombre de fonctions testées, le nombre de lignes \dots) puis parcourir le code du composant annoté du nombre de passages dans les fonctions au cours des tests.

`ng test --code-coverage`	avec karma
`jest -–coverage`		avec jest

Un dossier "coverage" est créé dans le projet. Il contient un fichier index.html qu'il faut ouvrir afin d'obtenir le rapport de couverture des tests.

## Commenter un test
`it() => xit()`

# Génération automatique de tests
Certains modules permettent de générer automatiquement des tests pour un composant. L’un deux s’appelle **ngentest**, et est disponible sur npm : https://www.npmjs.com/package/ngentest
ngentest génère des tests avec jest. Il est possible de générer des tests avec karma, mais il faut changer la configuration de ngentest.

**Installer** : `npm install ngentest –g`
**Générer des tests** : `ngentest *path/to/component.ts* `
**Générer des tests dans le fichier *path/to/component.spec.ts*** : `ngentest *path/to/component.ts* -s`

Ce que permet de générer ngentest :
-	Des mock de services, de classes et de pipe
-	Crée le Testbed avec les providers, schema et déclarations nécessaires (attention, il peut parfois manquer certains providers)
-	Un test pour chaque méthode afin de vérifier si la méthode est correctement appelée, ainsi que d’autres tests nécessaires.
ngentest ne permet pas de générer des tests plus complexes. Il faudra en créer de nouveaux pour tester plus efficacement le code. Cependant, sans plus de tests, la génération permet d’obtenir un bon coverage, avec souvent plus de 60% par composant.

# Passer de Karma à Jest
1) Installer Jest avec la commande: `npm install jest @types/jest --only=dev`
2) Installer jest-preset-angular: `npm install jest-preset-angular`
Puis créer un fichier **jest.config.js** contenant : 
```
const {
    pathsToModuleNameMapper
} = require('ts-jest/utils');
const {
    compilerOptions
} = require('./tsconfig');

module.exports = {
    preset: 'jest-preset-angular',
    roots: ['<rootDir>/src/'],
    testMatch: ['**/+(*.)+(spec).+(ts)'],
    setupFilesAfterEnv: ['<rootDir>/src/test.ts'],
    collectCoverage: true,
    coverageReporters: ['html'],
    coverageDirectory: 'coverage/my-app',
    moduleNameMapper: pathsToModuleNameMapper(compilerOptions.paths || {}, {
        prefix: '<rootDir>/'
    })
};
```
3) Ajouter la configuration de Jest dans **package.json**
Exemple de fichier **package.json**
```
{
 "name": "my-package",
 "version": "0.0.1",
 "license": "MIT",
 "scripts": {
  "test": "jest",
  "test:watch": "jest --watch",
  "test:cc": "jest --coverage"
 },
 "dependencies": {
  "@angular/common": "7.2.1",
  "@angular/compiler": "7.2.1",
  ...
 },
 "devDependencies": {
  "@types/jest": "^24.0.6",
  "jest": "^24.1.0",
  "jest-preset-angular": "^6.0.2",
  "ts-node": "~7.0.1",
  "typescript": "3.2.4"
 },
 "jest": {
  "preset": "jest-preset-angular",
  "setupTestFrameworkScriptFile": "<rootDir>/setupJest.ts"
 }
}
```
4) Remplacer les **ng test** par **jest**, notamment dans **tsconfig.spec.json**, et dans **tsconfig.json**:
```
"compilerOptions": {
    ...
    "types": ["jest"]
}
```
5) Enlever Jasmine et Karma: 
`npm uninstall jasmine @types/jasmine`
`npm remove karma karma-chrome-launcher karma-coverage-istanbul-reporter karma-jasmine karma-jasmine-html-reporter`

# TestBed
Chaque test de composant doit contenir un TestBed afin de gérer les injections de dépendances et le component binding.

Un exemple avec un composant ayant un @Input hero 
Dans le `beforeEach()` on écrit :
```javascript
TestBed.configureTestingModule({
	declarations : [HeroComponent]})
Fixture = TestBed.createComponent(HeroComponent)
```
Puis dans le test on peut utiliser le hero:
`fixture.componentInstance.hero = {…}`

## Providers
Dans le TestBed, il faut renseigner toutes les dépendances du composant (service ou classe), dans les « providers ». 
Exemple :
```javascript
providers: [ Router, Location, FormBuilder ]
```
Certaines dépendances nécessitent qu’on leur associe un mock. Soit parce qu’on va l’utiliser dans nos tests, soit parce que c’est une interface qui nécessite une implémentation explicite.
Pour utiliser un mock, la syntaxe est celle-ci : 
`{provide : ClassToMock, useClass : MockClass}`. 
Un exemple récurrent est celui du  Router :
Avant le testBed on définit notre mockRouter:
```javascript
class MockRouter {
  navigate() { };
}
```
Puis dans le TestBed:
`providers : [{ provide: Router, useClass: MockRouter }]`

/!\ Avec Jest, il y a souvent des problèmes avec la classe Location. Lorsque celle-ci doit être mise dans les providers, il faut ajouter également ceci :
```
{ provide: LocationStrategy, useClass: PathLocationStrategy },
{ provide: APP_BASE_HREF, useValue: '/app' }
```
## Schema 
Lorsqu’il y a un élément inconnu dans le HTML d’un composant, comme par exemple une balise mal écrite ou un attribut qui n’existe pas, une erreur s’affiche et le test ne passe pas. Cependant il peut être intéressant d’ignorer ces erreurs, par exemple lorsqu’une balise n’est pas une balise native HTML, mais un composant ou une balise d’un matériel Angular.

Lorsqu’un composant possède dans son HTML un appel à un composant fils, une erreur peut se déclencher avec les tests, car dans les tests on ne reconnait pas l’élément fils comme un élément HTML. Pour ne pas se préoccuper des éléments et attributs inconnus, il faut le mettre dans la déclaration du TestBed : `schema : [NO_ERRORS_SCHEMA]`

Cependant, lorsqu’on utilise des éléments Ionic, on aimerait que ceux-ci soient reconnus comme des éléments HTML, mais que s’il y a un problème dans le schéma HTML cela soit notifié. Pour résoudre ceci, il faut déclarer comme schéma `CUSTOM_ELEMENTS_SCHEMA`

# Tests basiques
## Les « matchers »
Jest utilise les matchers pour tester les valeurs de façon différentes.
Par exemple pour tester une égalité, on utilisera le matcher "toBe()" : expect(2 + 2).toBe(4).
Pour tester l'égalité d'un objet, on utilisera plutôt "toEqual()" : expect(data).toEqual({one: 1, two: 2}).
Pour tester la correspondance avec une expression régulière : expect('Christoph').toMatch(/stop/).
Pour tester si un Array contient bien un élément : expect(shoppingList).toContain('beer').
La liste de tous les matchers est disponible ici : https://jestjs.io/docs/en/expect

Configurer avant ou après les tests
Typiquement, lorsqu'on travaille avec une base de donnée (de test), on a besoin de l'initialiser avec de faire les tests. On peut vérifier qu'aucune donnée ne va corrompre nos tests, ou bien ajouter des données nécessaires initialement... Puis vider la base après les tests. Pour cela on utilise `beforeEach()` ou `afterEach()` avant ou après chaque test.
Si on a besoin de faire quelque chose avant tout test, on peut utiliser `beforeAll()` et `afterAll()`
Par exemple : 
```javascript
beforeEach(() => {
  initializeCityDatabase();
});

afterEach(() => {
  clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});
```
Par défaut, ces blocks before et after s'appliquent à tous les tests. Pour restreindre leur utilisation à certains tests, il faut utiliser `describe`.
```javascript
describe('matching cities to foods', () => {
  // Applies only to tests in this describe block
  beforeEach(() => {
    return initializeFoodDatabase();
  });

  test('Vienna <3 sausage', () => {
    expect(isValidCityFoodPair('Vienna', 'Wiener Schnitzel')).toBe(true);
  });

  test('San Juan <3 plantains', () => {
    expect(isValidCityFoodPair('San Juan', 'Mofongo')).toBe(true);
  });
});
```

# MOCK

Pour tester une fonction qui contient des dépendances, il est important de simuler les dépendances, afin de s'assurer que le test que l'on effectue sur une fonction, ne dépende que de la dite fonction.
Les fonctions "Mock" permettent ainsi de tester les liens entre le code en effaçant l'implémentation réelle d'une fonction, en capturant les appels à la fonction (et les paramètres transmis dans ces appels), en capturant les instances des fonctions du constructeur lors de l'instanciation avec New et en permettant une configuration en temps de test des valeurs de retour.
Il existe deux manières de simuler des fonctions: soit en créant une fonction Mock à utiliser dans le code de test, soit en écrivant un Mock manuel pour remplacer une dépendance de module.

Un exemple de mock de fonction: On a une fonction forEach qui prend en paramètre une liste, et un callback.
```javascript
function forEach(items, callback) {
  for (let index = 0; index < items.length; index++) {
    callback(items[index]);
  }
}
```
Un exemple de mock de fonction: On a une fonction forEach qui prend en paramètre une liste, et un callback.
```javascript 
let myMock = jest.fn();
myMock
  .mockReturnValueOnce(10)
  .mockReturnValueOnce('x')
  .mockReturnValue(true);
  ```
`console.log(myMock(), myMock(), myMock(), myMock());` renvoie 10, 'x', true, true. En effet on a mocké une fonction qui renvoie au premier appel 10, au deuxième appel 'x', et sinon true.
Un exemple de mock de module:
Nous avons une classe Users qui renvoie des utilisateurs d'une API. Cette classe utilise le module axios pour appeler l'API et renvoyer les données.
```javascript
import axios from 'axios';

class Users {
  static all() {
    return axios.get('/users.json').then(resp => resp.data);
  }
}
```

Nous voulons tester notre classe sans réellement appeler l'API à l'aide de axios. Pour cela nous utilisons jest.mock() pour mocker le module axios.
```javascript
import axios from 'axios';
import Users from './users';

jest.mock('axios');

test('should fetch users', () => {
  const users = [{name: 'Bob'}];
  const resp = {data: users};
  axios.get.mockResolvedValue(resp);

  // or you could use the following depending on your use case:
  // axios.get.mockImplementation(() => Promise.resolve(resp))

  return Users.all().then(data => expect(data).toEqual(users));
});
```
## Mocking Injected Services
Dans TestBed : `providers:[{provide: HeroService, useValue: mockHeroService}]`
On mock le service avec un spy jasmine pour Karma: 
`mockHeroService = jasmine.createSpyObj([‘getHeroes’, ‘addHero’, ‘deleteHero’])`
Ou un spy jest : Il y a deux manières de mocker une fonction en utilisant jest : soit simplement avec `jest.fn()`, soit avec `jest.spyOn(object, methodName)`. `spyOn` crée un mock de fonction similaire à `jest.fn`, mais permet de traquer les appel à `object[methodName]`. Ainsi avec spyOn on pourra facilement utiliser le test `expect(spy).toHaveBeenCalled()`.
`mockDeleteHero = jest.spyOn(HeroService, ‘deleteHero’)`

Si dans le code d’une méthode on s’abonne à un service ou une méthode, pour pouvoir mocker ce service ou méthode, il faut retourner un observable : `.and.returnValue(of( ... ))`. Cela peut arriver notamment dans un composant CRUD : la méthode delete du composant va souscrire à un service permettant de supprimer un enregistrement. La valeur de l’observable retournée a souvent peu d’importance. On peut mettre simplement « true ». 
`mockHeroService.deleteHero.and.returnValue(of(true))`

## Mocking child components
Pour mocker un composant enfant, on crée une fausse classe. Il faut ajouter la déclaration de ce faux composant dans le test:
```javascript
@Component({selector :’app-hero’, template : ‘<div></div>’})
export class FakeHeroComponent { @Input() hero : Hero}
```
Puis dans la partie **‘déclaration’** du TestBed : `declarations : [FakeHeroComponent]`.

## Mocking http request to an API
Ajouter dans les imports du TesBed `HttpClientTestingModule`
Ecrire: `httpTestingController = TestBed.get(HttpTestingController)`
Puis dans les tests :
```javascript
const req = httpTestingController.expectOne(‘api/hero/4’)
req.flush({id:4, name: ‘Super’, strength:100})
```

## Mocking Router
Créer une classe pour mocker le routeur:
```javascript
class MockRouter {
        navigate = jest.fn();
    }
```
Puis dans le TestBed:
`providers: [{ provide: Router, useClasse: MockRouter }`

## Mocking ActivatedRoute
Ecrire avant le TestBed:
`const mockParams = new Subject<Params>();`
Puis dans le TestBed:
`providers: [{ provide: ActivatedRoute, useValue: { params: mockParams } }]`
OU 
`{ provide: ActivatedRoute, useValue: { params: of({ text: 'texte' }) } }` quand on veut réutiliser le params facilement

Pour utiliser les params dans les tests, on utilise `next()`:
```javascript 
mockParams.next({‘id’ : ‘123’})
fixture.detectChanges()
```

## Mocking AlertController
Créer les classes de Mock:

```javascript
class MockAlert {
    public visible: boolean;
    public header: string;
    public message: string;

    constructor(props: any) {
        Object.assign(this, props);
        this.visible = false;
    }

    present() {
        this.visible = true;
        return Promise.resolve();
    }

    dismiss() {
        this.visible = false;
        return Promise.resolve();
    }

}

class MockAlertController {

    public created: MockAlert[];

    constructor() {
        this.created = [];
    }

    create(props: any): Promise<any> {
        const toRet = new MockAlert(props);
        this.created.push(toRet);
        return Promise.resolve(toRet);
    }

    getLast() {
        if (!this.created.length) {
            return null;
        }
        return this.created[this.created.length - 1];
    }
}
```
Dans le testbed écrire le provider
`providers: [{provide: AlertController, useValue: new MockAlertController()}]`

Puis utiliser dans le test:
```javascript
const mockAlertController = component.alertController as any as MockAlertController;
expect(mockAlertController.getLast().visible).toBeTruthy();
expect(mockAlertController.getLast().header).toBe('Test');
expect(mockAlertController.getLast().subHeader).toBe('Hola html.userName');
```

# Manipuler les éléments
## Testing render HTML
Pour récupérer un élément à l'aide d'un sélecteur:
`fixture.nativeElement.querySelector(…)	`	ou 	`fixture.debugElement.query(By.css(…)).nativeElement`
/!\ Attentio, pour détecter le binding il faut ajouter ensuite : `fixture.detectChanges()`

##Finding elements by directive
`fixture.debugElement.queryAll(By.directive( … ))`
Exemple de directive : `HeroComponent`pour récupérer les HeroComponent en tant que debug elements.

## Triggering Events on Elements
Exemple : on mock la méthode delete du composant. On mock le service HeroService afin qu’à l’appel de ‘getHeroes’, il renvoie des observables de HEROES. On trigger l’évenement click au clic sur le bouton d’un Hero. Enfin on vérifie que la méthode delete a bien été appelée avec le bon hero.
```javascript
mockDelete = jest.spyOn(fixture.componentInstance, ‘delete’) ;
mockHeroService.getHeroes.and.returnValue(of(HEROES));
fixture.detectChanges();
const heroComponents = fixture.debugElement.queryAll(By.directive(HeroComponent));
heroComponents[0].query(By.css(‘button’)).triggerEventHandler(‘click’, {stopPropagation: () => {}});
expect(mockDelete).toHaveBeenCalledWith(HEROES[0]);
```

## Emit events from children
`(<HeroComponent> heroComponents[0].componentInstance).delete.emit(undefined)`

## Raising events on child directives
`heroComponents[0].triggerEventHandler(‘delete’, null)`

# Code asynchrone

## Méthode 1: utiliser le setTimeout
```javascript
setTimeout(() => { expect (mockHeroService.updateHero).toHaveBeenCalled()}, 300) ;
OU
it(‘…’, (done) => {
	…
	setTimeout(() => { 
		expect (mockHeroService.updateHero).toHaveBeenCalled();
		done();
	}, 300) ;
})
```

## Méthode 2 : Utiliser fakeAsync
```javascript
it(‘…’, fakeAsync (() => {
	…
	tick(250)		OU 	flush();
	expect (mockHeroService.updateHero).toHaveBeenCalled();
}))
```

## Méthode 3: Utiliser async
```javascript
it(‘…’, async (() => {
	…
	fixture.whenStable().then( () => {
		expect (mockHeroService.updateHero).toHaveBeenCalled();
	})
})
```


