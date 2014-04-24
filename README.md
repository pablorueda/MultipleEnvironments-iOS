# Multiple Environments Configuration

## Steps

Short step list to create a project with different environments.

1. Set up project configurations.
2. Create environment bundles with a Setting.plist.
3. Add a build setting with the path to the bundle.
4. Add the bundle copy script to the build phase.
5. Don't add the bundles to the target membership.
6. Create the Settings class that reads the plist.
7. Set up project scheme.

## Full tutorial

These are the steps to create a project with 3 environments: development, preproduction and production.

1 We need to change the current configurations of the project (usually Debug and Release) for our 3 environments. With the project selected select Project -> Info -> Configurations, change the name of Debug for Development, rename Release for Production, and duplicate it and name it Preproduction.

2 Add a bundle named Development to the project and inside it create a property list called Settings. In this property list we will add all the variables dependent of the environment. In our case a string named SERVER_URL with an url. Repeat this process for each environment, changing the bundle name.

3 We need to add a User-Defined-Setting called ENVIRONMENT_BUNDLE. With the project selected and the Build Settings tab selected, click Editor -> Add Build Setting -> Add User-Defined Setting and name the setting.
Write the path of the bundle for each environment. In our case it will be: 

	${PROJECT_DIR}/MultipleEnvironments/Development.bundle
	${PROJECT_DIR}/MultipleEnvironments/Preproduction.bundle
	${PROJECT_DIR}/MultipleEnvironments/Production.bundle

4 We created 3 bundles for each environment, but we want that our application treat them as just one. For doing that, in compilation time we will copy the current bundle and rename it. With the target selected and the Build Phases tab selected, click Editor -> Add Build Phase -> Add Run Script Build Phase. And write the following script

	cp -r "${ENVIRONMENT_BUNDLE}" "${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app/Environment.bundle"

5 As our final bundle is copied in the final app, we no longer need the bundles in our target. For each bundle deselect the Target Membership in the File Inspector. If we don't do this step, our final application will be deployed containing all our development settings and any user could see it.

6 Now we need to create the class that the rest of our code will use to access to the environment variables. Create a class called Settings. This class will contain properties with the variables in settings.plist. 
We will need to access to the bundle path, after to the plist inside that bundle, convert it to a dictionary, and read all the variables.
In the header file we will have the properties:

	@property (copy, nonatomic) NSString *serverURL;
	
And in the init method we will set them:

	NSString *settingsBundlePath = [[NSBundle mainBundle] pathForResource:@"Environment" ofType:@"bundle"];
	NSBundle *settingsBundle = [NSBundle bundleWithPath:settingsBundlePath];
	NSString *settingsPListPath = [settingsBundle pathForResource:@"Settings" ofType:@"plist"];
	NSDictionary *settings = [[NSDictionary alloc] initWithContentsOfFile:settingsPListPath];
	_serverURL = [settings valueForKey:@"SERVER_URL"];

7 The last step is to configure our scheme to set up the desired enviroment  when we run, test or archive. Click Product -> Scheme -> Edit Scheme and in Build Configuration change to the environment you desire when you run the app. Do the same for test, archive, etc.

That's all.

Now we can access to the setting class and read the properties as we will do normally. I recommend to have a unique instance of the Setting class, with a singleton for example, to don't need to parse the plist every time we need to read a property.

## Additional considerations

### Plist instead of a bundle

We created a bundle for each environment and added the Setting.plist inside it. If you only need different variables for the environment, you can remove the bundle and set just the property list. However I recommend to use a bundle as you can place inside resources that are different depending on the environment, as certificates, launch images, etc.

### Different bundle, app name… for each environment

We can set a different app name for each environment adding an User-Defined Build Setting and writing a different value for each configuration. 

	APP_NAME -> Dev-Name
	
After with the target selected and the info tab selected, we can change the bundle display name property and assign it our custom setting.

	bundle display name  -> ${APP_NAME}