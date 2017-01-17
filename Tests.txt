package com.retailmenot.testcases;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;
import java.util.concurrent.TimeUnit;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.ie.InternetExplorerDriver;
import org.testng.Assert;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Parameters;

import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

import com.retailmenot.pages.HomePage;
import com.retailmenot.pages.ProductsDealsPage;
import com.retailmenot.util.AppLibrary;


import jxl.read.biff.BiffException;

public class RetailMeNotTests 

{
	Properties prop;
	WebDriver driver;
	HomePage hPage;
	ProductsDealsPage dealspage;
	String url;
	AppLibrary appLib;
	
	@Parameters({"browserType","url"})
	@BeforeClass
	public void invokeBrowser(String browserType,String url) throws IOException
	{
		FileInputStream fis = new FileInputStream(new File("RetailMeNot.properties"));
		prop = new Properties();
		prop.load(fis);
		
		if(browserType.equals("FF"))
		{
		
			  driver = new FirefoxDriver();
		}
		else if(browserType.equals("IE"))
		{
			System.setProperty("webdriver.ie.driver","IEDriverServer.exe");
			driver = new InternetExplorerDriver();
		}
		else if(browserType.equals("CH"))
		{
			  System.setProperty("webdriver.chrome.driver", "chromedriver.exe");
			  driver  = new ChromeDriver();
		}
		
		
		hPage = new HomePage(driver,prop);
		this.url = url;
		appLib = new AppLibrary();
		
}
	@Test(dataProvider="DP")
	public void verifySearchResult(String categoryName,String itemCount) throws InterruptedException
	{
		driver.get(url);
		driver.manage().window().maximize();
		driver.manage().deleteAllCookies();
		driver.manage().timeouts().implicitlyWait(150, TimeUnit.SECONDS);
		Thread.sleep(10000);
		dealspage = hPage.browseCoupons("Product Deals");
		boolean result = dealspage.verifyTitle("Product Deals");
		Assert.assertTrue(result);
		dealspage.jumpToCategory(categoryName);
		int actual = dealspage.retriveCount(categoryName);
		int expected = Integer.parseInt(itemCount);
		Assert.assertEquals(actual, expected);
		
		
	}
	
	@DataProvider(name="DP")
	public String[][] readExcelData() throws BiffException, IOException
	{
		String inputData[][]= appLib.readExcel("CategoryData.xls");
		return inputData;
	}
}
