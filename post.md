QA Automation at Global Personals
=================================
I joined the Global Personals Development Team as a developer in test in September 2013. Global had successfully started an automation testing programme for a year before I joined. During the last eleven months, we've made quite a lot further improvement to our framework as well as massively increasing our test library. So here I would like to review what we have achieved and what I have learnt about automation.

### Framework
We use ruby to develop the automation scripts, just as most of our ruby apps. Therefore we use RSpec as the test framework, plus Capybara and Selenium for all the web application tests.

#### Page Object
Previously to reduce the duplicated codes, we had created common helpers. However, after a few months more helpers were added, it caused problems that we have no idea where to find/add our helpers. With our research and discussion we then decided to use Page Object patter to build our own DSL. The idea is to encapsulate the page's behaviours into the page object, which is good for maintaining automation and increasing code readability. Below is an example of login page object

```
class LoginPage
  def login_as(member)
    fill_in :memberid, with: member.email
    fill_in :password, with: member.password
    find("button[type='submit']").click
  end
end
```

Call `login_page.login_as` in the automation scripts can log the member in, easy and readable. It's giving a clear code structures where to find/add the helpers. Another benifit is we can now guess the methods in automation. If would like to perform a payment with automation, try to find `pagement_page`.

### Continuous integration
As part of BDD, the end-to-end automation scripts are able to guarantee the correct behaviours of our application, no matter any changes made in the application. With our success in writing automation scripts, naturally we moved forward to CI. By building our CI system with [Jenkins](http://jenkins-ci.org/), we can successfully run automation scripts against different environments and branches. The CI results will be output in HTML on the sever and be posted to our team communication app.

But that's not enough yet. As a good tester we'd like to provide the steps to replicate the issue and the clear result of the failure, so we added the ability to automation to take screenshots when example is failing, and attached to the HTML. People without any knowledge of qa automation can read the CI results and no need to imagine.

So far we've managed to run 100+ end-to-end tests within an hour in our CI system (it's all running in one virtual box, amazing!), and it's proven that CI is able to capture bugs in our day-to-day work!

### Stability
Since most of our automation scripts are End-to-End tests and our product has a lot of user interfaces, there are lots of web UI tests in the test suite. As a result, we experiences many intermittent issues like request timeout, page is not completely loaded, server is down etc. These reduce the confidence of our automation tests. Therefore, we need some techniques to help the driver to deal with them in order to increase the tests stability.

To handle the timeout or element not found, waiting is good solution. There are two types of wait, implicit wait and explicit wait.

##### Implicit wait
Simply use `Capybara.default_wait_time` will change the capybara implicit wait time. Capybara is smart enough to retry finding the link for a brief period of time before giving up and throwing an error. However, change the golobal default wait time means automation is slowed down. Thankfully Capybara allows to temporarily change the time by using `Capybara.using_wait_time() {}`

##### Explicit wait
As the name, we tell Capybara to sleep until the element is there. The code will be like `sleep 1 until page.has_css?('#targetElementID')` This is better than implicit wait as we don't have to wait up to the default wait time when we expect not to have content on the page. But we also need to be careful with this wait. As if the sever is down, the automation will dead in that infinite loop. So we would add `Timeout::timeout() {}` to the loop.

### Speed up
Last but not least, the speed of automation. In the agile process, we'll need to add more end-to-end tests to CI as a result of new feature required. But we don't want to slow down the CI as in theory, a CI run should finish in an hour as the agile test. We don't have unlimited budget to have thousands VMs. Therefore we need some tools / techniques to speed up.

The first idea is to use headless driver. Selenium is good while developing automation so you can watch it. CI doesn't really need to open a browser. We choose [poltergeist](https://github.com/teampoltergeist/poltergeist) as the headless driver, it doesn't load the entire browser and can simulate the user activities. It increases 20% of the automation speed, amazing.

### Repeat tests
Because we are developing with selenium driver, it's possible that some cases are not working with poltergeist driver. In order to get the balance of speed and accuracy, we built a repeat mechanism in our automation. The automation scripts will be executed concurrently with poltergeist driver, the failing examples are recorded in `rspec.failures` file (if any), then CI will use selenium to re-run just those failing examples.

### Summary
Here in Global we've made great progress in test automation, it speeds regression test while not reducing the test confident level, really helping software development in agile approach. Of course there are still lots of works can be done in automations. E.g. catch the JavaScript errors on the page then send the whole error stack to the developers. We will never stop improving the automation because it's really enjoyable and helpful! 

If you would like to work with us in global, check out our jobs page, [we're hiring](http://globaldev.co.uk/jobs/)
