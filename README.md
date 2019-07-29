# Navigation-Component-OptionsMenu-with-Custom-Transition-Animation-and-Custom-Navigation-Listener
This Tutorial will discuss how to implement custom transition animation when using Navigation Component provided by Android Jetpack and OptionsMenu.

# Prerequisite:
This tutorial assumes that you are familiar with the following:
1. Android Jetpack Navigation Component
2. OptionsMenu

# Discussion
If you are familiar with Android Jetpack Navigation Component, you will find that it's too easy to combine your navigation graph with a OptionsMenu. You will only use two lines of code to make this done as stated in the Android Documentation.

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        NavController navController = Navigation.findNavController(this, R.id.nav_host_fragment);
        return NavigationUI.onNavDestinationSelected(item, navController) || super.onOptionsItemSelected(item);
    }

As you can see here, there is no much control for you over the way of communication between the NavController and OptionsMenu. The NavigationUI class will handle everything for you starting with making the navigation when you press on a menu item, applying default animation for transition.



# But What if you want to Apply Custom Transition Animations
For this case scenario, you may have 3 menu items in the OptionsMenu. Two of them is used to navigate between available destinations and the last one is used to start a new activity (maybe a SettingsActivity). What will come up on your mind is to use the onOptionsItemSelected() method as stated in the Android Documentation. But using the NavigationUI.onNavDestinationSelected() method won’t allow us to add custom transition animations.  

        NavigationUI.onNavDestinationSelected(item, navController);

Here is the Steps to achieve the desired behavior in this case scenario;
1. Create a NavOption object and pass the custom transition animation to it using the following methods

                .setEnterAnim(R.anim.slide_in_right)
                .setExitAnim(R.anim.slide_out_left)
                .setPopEnterAnim(R.anim.slide_in_right)
                .setPopExitAnim(R.anim.slide_out_left)
        
2. You have to make sure of creating only one instance of the same destination target (the destination fragment you are navigating to) by adding this parameter to the NavOption Object

                .setLaunchSingleTop(true)
                
3. You will also need to make sure that the navigation BackStack is cleared with every transition so that when you press the back button, you go for the start destination fragment, not the previous selected fragment. This can be achieved by adding the following parameters to the NavOption Object

                .setPopUpTo(navController.getGraph().getStartDestination(), false)

4. Pass the NavOption Object to the NavController.navigate() method to be applied for the navigation operation
                    navController.navigate(item.getItemId(), null, navOptions);

5. Override the onOptionsItemSelected() method to control what happen when you select a menu item
6. Check if the NavGraph contains a similar destination as the menu item id
7. For menu items that used to navigate between the NavGraph destination, Use 
navController.navigate(item.getItemId(), null, navOptions) to handle the navigation for you.
8. If there is no such destination, Use Switch/Case block to differentiate between the menu items

Here is a complete code snippet

    NavController navController;
    NavOptions navOptions;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ...

        navController = Navigation.findNavController(this, R.id.nav_host_fragment);

        navOptions = new NavOptions.Builder()
                .setLaunchSingleTop(true)
                .setEnterAnim(R.anim.slide_in_right)
                .setExitAnim(R.anim.slide_out_left)
                .setPopEnterAnim(R.anim.slide_in_right)
                .setPopExitAnim(R.anim.slide_out_left)
                .setPopUpTo(navController.getGraph().getStartDestination(), false)
                .build();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        this.optionsMenu = menu;
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.menu, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        if (navController.getGraph().findNode(item.getItemId()) != null) {
            navController.navigate(item.getItemId(), null, navOptions);
        } else {
            switch (item.getItemId()) {
                case R.id.settings:
                    openSettingsActivity();
                    break;
            }
        }
        return super.onOptionsItemSelected(item);
    }

