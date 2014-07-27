QA Automation at Global Personals
=================================

Hi there, my name is Joe Xie and I joined the Global Personals Development Team as a developer in test in September 2013. Global have successfully started automation tests for a year before I joined and during the last ten months, we also made quite a lot progresses in test automation. So here I would like to review what we have achieved and what I have learnt about automation.

### Framework
Our automation scripts are written in Ruby and RSpec is our test framework to write automation tests. As most of our product are web application, we also use capybara to simulate real users interact with the application. Furthermore, capybara combines with selenium driver allows us to run automation scripts with modern browsers such as Firefox, Chrome and IEs. It's worthing nothing that we don't use cucumber in our test automation. Because it could easily increase the complexity such as parsing the keywords with regular expression. Instead we create our own DSL can make the specs clean and readable.

#### Page Object
To build our DSL, we use Page Object Pattern. The idea is to encapsulate the page's behaviours into the page object, which is good for maintaining automation and increasing code readability. Below is an example of login page object

```
class LoginPage
  def login_as(member)
    fill_in :memberid, with: member.email
    fill_in :password, with: member.password
    find("button[type='submit']").click
  end
end
```

Also with the magic of `method_missing` in ruby, we can easily initialise page object and add more page objects.

```
def method_missing(method_name, *args, &block)
  if method_name =~ /page$/
    if instance_variable_defined?("@#{method_name}")
      instance_variable_get("@#{method_name}")
    else
      instance_variable_set("@#{method_name}", const_get("#{class_name}").new(self, *args))
    end
  else
    super
  end
end
```

E.g. we can call login page object in a Capybara Session like `login_page.login_as`, it will then initialise the page object dynamically.

### Continuous integration
As part of BDD, the end-to-end automation scripts are able to guarantee the correct behaviours of our application, no matter any changes made in the application. With our success in writing automation scripts, naturally we moved forward to CI. By building our CI system with [Jenkins](http://jenkins-ci.org/), we can successfully run automation scripts against different environments and branches. The CI results will be output in HTML on the sever and be posted to our team communication app.

We've managed to run 100+ end-to-end tests within 40 minutes in our CI system (it's all running in one virtual box, amazing!), and it's proven that CI is able to capture bugs in our day-to-day work!

### Stability
CI is not the end of the test automation. Just like many other programmes, automation has its own difficulty. Since most of our automation scripts are end-to-end tests, there are lots of web UI tests in the scripts. As a result, we experience many intermittent issues like request timeout, page is not completely loaded, server is down etc. It will destroy the confidence of tests because you'll have no idea if it's a bug or just environmental. Therefore, we will need some techniques to help driver to to increase the stability of automation.

To handle the timeout or element not found, waiting is good solution. There are two types of wait, implicit wait and explicit wait.

##### Implicit wait
Simply use `Capybara.default_wait_time` will change the capybara implicit wait time. Capybara is smart enough to retry finding the link for a brief period of time before giving up and throwing an error. However, if you have an assertion to expect page not to have content, Capybara will wait up to the default wait time, it will slow down the automation.

##### Explicit wait
As the name, we tell Capybara to sleep until the element is there. The code will be like `sleep 1 until page.has_css?('#targetElementID')` This is better than implicit wait as we don't have to wait up to the default wait time when we expect not to have content on the page. But we also need to be careful with this wait. As if the sever is down, the automation will dead in that infinite loop. So we would add `Timeout::timeout() {}` to the loop.

### Speed up
Last but not least, the speed of automation. In the agile process, we'll need to add more end-to-end tests to CI as a result of new feature required. But we don't want to slow down the CI as in theory, a CI run should finish in an hour as the agile test. We don't have unlimited budget to have thousands VMs. Therefore we need some tools / techniques to speed up.

The first idea is to use headless driver. Selenium is good while developing automation so you can watch it. CI doesn't really need to open a browser. We choose [poltergeist](https://github.com/teampoltergeist/poltergeist) as the headless driver, it doesn't load the entire browser and can simulate the user activities. 

Another idea is to use [parallel_tests gem](https://github.com/grosser/parallel_tests). It can speed up RSpec by running parallel on multiple CPUs (or cores). With our experiment, one of our end-to-end test is taking 10 minutes normally. By using parallel_test it reduces to 8 minutes.

### Summary
Here in Global we've made great progress in test automation, it speeds regression test while not reducing the test confident level, really helping software development in agile approach. Please let me know if there’s anything missing or incorrect here.
