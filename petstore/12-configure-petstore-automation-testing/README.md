
# 12 - Configure Azure DevOps pipelines to execute automated regression tests

__This guide is part of the [Azure Pet Store App Dev Reference Guide](../README.md)__

In the previous guides we've looked at the steps needed to deploy our live Pet Store Application which can be found here [https://azurepetstore.com/](https://azurepetsore.com/). However, we haven't yet looked at automated regression testing and how that fits into this application. Suppose there is a QA team responsible for testing this Pet Store Application.  Perhaps this team is responsible for writing end to end tests which smoke test the application, perform synthetic monitoring and/or verify the UI and all of the functionality of the web application and down stream service(s) is in fact working as expected.  This guide will look at the steps needed to write tests (using Selenium & JUnit) and execute upon successful deployment of the Pet Store Application, acting as the automated regression suite after each and every deployment. For thus we will create a separate project that will trigger independently of the production Pet Store Application code.

> 📝 Please Note,  this automation tests can be written in many different languages, the objective of this guide is to walk through the steps needed to execute your tests from an Azure DevOps Pipeline upon success of an application deployment, You can also execute these regression tests on a schedule if you prefer, for example, you may want to execute daily/hourly etc... to ensure your application is performing as expected, all of which is possible with Azure DevOps as well. For the sake of this guide, we will be triggering upon success of an application deployment.

**Step 1**
The following petstoreautomation project https://github.com/chtrembl/azure-cloud/tree/main/petstore/petstoreautomation contains the tests needed for Pet Store Application. If you inspect https://github.com/chtrembl/azure-cloud/blob/main/petstore/petstoreautomation/src/test/java/petstore/automation/AzurePetStoreAutomationTests.java you will see the following:

```java
package petstore.automation;

import static org.junit.Assert.assertEquals;

import org.junit.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.htmlunit.HtmlUnitDriver;

/**
 * Automation Tests for Azure Pet Store
 */
public class AzurePetStoreAutomationTests {

	private HtmlUnitDriver unitDriver = new HtmlUnitDriver();

	private String URL = "https://azurepetstore.com/";

	private String DOG_BREEDS = "Afador,American Bulldog,Australian Retriever,Australian Shepherd,Basset Hound,Beagle,Border Terrier,Boston Terrier,Bulldog,Bullmastiff,Chihuahua,Cocker Spaniel,German Sheperd,Labrador Retriever,Pomeranian,Pug,Rottweiler,Shetland Sheepdog,Shih Tzu,Toy Fox Terrier";

	@Test
	// Test the Azure Pet Store App Title
	public void testAzurePetStoreTitle() {
		this.unitDriver.get(this.URL);
		System.out.println("Title of the page is -> " + this.unitDriver.getTitle());
		assertEquals("Azure Pet Store", this.unitDriver.getTitle());
	}

	@Test
	// Test the Azure Pet Store App Dog Breed Page and Downstream Azure Pet Store
	// Service / Dog Breed API
	public void testAzurePetStoreDogBreedCount() {
		this.unitDriver.get(this.URL + "dogbreeds?category=Dog");
		WebElement element = this.unitDriver.findElement(By.className("table"));
		String dogBreedsFound = new String(element.getText()).trim().replaceAll("\n", ",");
		System.out.println("Dog Breeds found in the page -> " + dogBreedsFound);
		assertEquals(this.DOG_BREEDS, dogBreedsFound);
	}
}
```
Using the Junit framework, we can run assertions on various scenarios and HtmlUnitDriver will allow us to execute web requests in a headless browser and easily inspect HTML. For the sake of this guide, I have written two tests:

**testAzurePetStoreTitle**
**testAzurePetStoreDogBreedCount**

Both of these tests will assert (true/false) based on the condition that has been written. 

**testAzurePetStoreTitle** will assert that the https://azurepetstore.com homepage returns with an expected title.

**testAzurePetStoreDogBreedCount** will assert that the https://azurepetstore.com/ page will return with the expected 20 dog breeds, which effectively asserts that the application is running along with the downstream service that is responsible for providing the dog breeds.

> 📝 Please Note, ideally you would have a full suite of tests here meeting all of your business acceptance test criteria etc...
 
We could certainly ask maven to execute these tests locally

```
mvn clean test
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running petstore.automation.AppTest
Title of the page is -> Azure Pet Store
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 11.34 s - in petstore.automation.AppTest
```
However, we can take this further by way of automation and trigger these automagically.

**Step 2**
Install the Trigger Plugin within Azure DevOps

> 📝 Please Note, 
You should see something similar to the below image:

![](images/1.png)

Things you can now do now with this guide

☑️ Configure Azure DevOps pipelines to execute automated regression tests