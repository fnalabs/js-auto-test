# js-auto-test
The purpose of `js-auto-test` is to provide some scaffolding on top of [protractor](http://www.protractortest.org/) to follow a Page Object pattern for Automated UI tests. The Library contains some classes to help structure and automate your tests. This is developed in parallel with the [docker-auto-test-starter](https://github.com/aeilers/docker-auto-test-starter) kit as the library that the Docker implementation is built upon. The starter kit has more of a complete example than what is provided below.

## Library
The library consists of two main classes: [Fragment](./src/js/Fragment.js) and [Sequence](./src/js/Sequence.js).

### Fragment
A Fragment is a reusable group of HTML element references that can be tested. For instance, a top level navigation bar is a reusable group of HTML elements that could show up on many pages. It can be used as a shared Fragment component that can be associated with other Fragments. If you have unique content on the home page, you can make a home page Fragment that is associated with your navigation Fragment above. The purpose of a Fragment is for testing its elements and optionally performing actions against its elements.

Fragment provides basic testing functionality for getting/setting elements stored in a Map. It also provides some basic test methods to test any child fragments as well as check if the elements exist on the page. To perform more complex tests, extend the functionality of the class with additional test methods as needed. Check the [testExists](./src/js/Fragment.js#L42) method for an example implementation. Don't forget to override [testElements](./src/js/Fragment.js#L35) to call your new methods after calling `await super.testElements()` to run the provided test methods.
- ***NOTE:*** Action methods/classes will need to be added per use case, [requests](https://github.com/aeilers/js-auto-test/issues) for common Actions are welcome. ([element api](http://www.protractortest.org/#/api?view=ElementFinder), [element.all api](http://www.protractortest.org/#/api?view=ElementArrayFinder))

### Sequence
A Sequence defines the steps an automated UI test specification needs to perform. It is also responsible to setting the entry point to the test sequence. It provides a Fragment cache to reference for each step in the sequence that will need to be defined.

I've started out with some basics and will be adding more over time (and open to [feature requests](https://github.com/aeilers/js-auto-test/issues)).

## Example
Here is a simple example of an implementation using `js-auto-test`. When testing a larger site with many tests, you will want to consider some structure around your code. I've added a suggested minimal folder structure below. I have also created a [starter kit](https://github.com/aeilers/docker-auto-test-starter) that contains the basic structure below with additional support scripts and environment/execution specifics.

#### Code
- `./constants.js`
  ```javascript
  // Selectors
  export const IMG_SELECTOR = '#hplogo';

  // Fragments
  export const GOOGLE_FRAGMENT = Symbol('google fragment');
  ```

- `./fragments/GoogleFragment.js`
  ```javascript
  import { IMG_SELECTOR } from '../constants';

  import { Fragment } from 'js-auto-test';


  export default class GoogleFragment extends Fragment {

      constructor(fragments) {
          super(fragments);

          this.setElement(IMG_SELECTOR);
      }

  }
  ```

- `./sequences/GoogleSequence.js`
  ```javascript
  import { GOOGLE_FRAGMENT } from '../constants';

  import { Sequence } from 'js-auto-test';

  import GoogleFragment from '../fragments/GoogleFragment';


  export default class GoogleSequence extends Sequence {

      constructor() {
          super();

          this.setFragment(GOOGLE_FRAGMENT, new GoogleFragment());

          this.setStep(() => this.getUrl('/'));
          this.setStep(this.getFragment(GOOGLE_FRAGMENT).testElements);
      }

  }
  ```

- `./specs/google.spec.js`
  ```javascript
  import GoogleSequence from '../sequences/GoogleSequence';

  browser.ignoreSynchronization = true;


  describe('google homepage img test', () => {
      let googleSequence;

      before(() => {
          googleSequence = new GoogleSequence();
      });

      it('expects img to exist on the google homepage', async () => {
          await googleSequence.runSequence();
      });

      after(() => {
          googleSequence = null;
      });
  });
  ```

#### Config
- `./conf/config.js`
  ```javascript
  exports.config = {
      directConnect: true,
      capabilities: {
          browserName: 'chrome',
          platform: 'ANY',
          version: ''
      },
      baseUrl: 'https://www.google.com',
      framework: 'mocha',
      mochaOpts: {
          reporter: 'spec',
          timeout: 5000
      },
      specs: ['../dist/**/*spec.js']
  };
  ```
  - ***NOTE:*** This configuration was used to run protractor on Linux Mint 18.1.

## Future
- unit tests!
- additional functionality for common tests and actions
  - v1.1 is planning to add a common Action class to define various user action support
- feature requests via [issues](https://github.com/aeilers/js-auto-test/issues)
