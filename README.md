# Multiple Environments Configuration

Steps to create a project with 3 environments: development, preproduction and production.

1 Project->Info->Configurations change the currents configurations (usually Debug and Release) for the configurations you want. In out case I will change the name of Debug for Development, and i will duplicate the Release, and rename both for PreProduction and Production.

2 For each enviroment add to the project a bundle and inside the bundle create a property list. In this property list (called Settings) we will add all the variables dependant of the environment. In our case a string for the URL server.

3 Add a User-Defined-Setting called ENVIRONMENT_BUNDLE. With the project selected: Editor->Add Build Setting->Add User-Defined Setting.

4 Write the path of the bundle for each environment. In our case for development it will be: ${PROJECT_DIR}/MultipleEnvironments/Development.bundle

5 We created 3 bundles for each environment, but we want that our application treat them as if it was just one. For doing that in compilation time we will copy the desired current bundle and rename it. With the target selected and the Build Phases tab selected: Editor->Add Build Phase->Add Run Script Build Phase.
Write the following script

	cp -r "${ENVIRONMENT_BUNDLE}" "${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app/Environment.bundle"

6 As our final bundle is copied in the final app, we no longer need the bundles in out target. For each bundle deselect the Target Membership in the File Inspector. If we don't do this, our final application will be deployed containing all our development settings and any user with enough knowledge could see it.

7 Create a class called Settings. This class will contain  properties with the variables in settings.plist. 
We will need to access to the bundle path, after to the plist inside that bundle, convert it to a dictionary and read all the variables.
In the header file:

	@property (copy, nonatomic) NSString *serverURL;
	
In the init method we will set these properties:

	NSString *settingsBundlePath = [[NSBundle mainBundle] pathForResource:@"Environment" ofType:@"bundle"];
	NSBundle *settingsBundle = [NSBundle bundleWithPath:settingsBundlePath];
	NSString *settingsPListPath = [settingsBundle pathForResource:@"Settings" ofType:@"plist"];
	NSDictionary *settings = [[NSDictionary alloc] initWithContentsOfFile:settingsPListPath];
	_serverURL = [settings valueForKey:@"SERVER_URL"];

8 The last step is to configure our scheme to set up the enviroment we desire when we run, test, archive. 
Product->Scheme->Edit Scheme and in Build Configuration change to the environment you desire when you run the app.

That's all.

Now we can access to the setting class and read the properties as we will do normally.
