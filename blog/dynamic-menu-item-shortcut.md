% Custom Keyboard Shortcuts to Dynamic Menu Items

Here, I outline a method for adding a keyboard shortcut to menu items with dynamic names in Mac OS X. Custom keyboard shortcuts on the Mac are generally great, and you can create your own system-wide, or application-specific shortcuts in the keyboard preference pane. All you need to do is provide the <strong>precise</strong> name of the menu item you want to make a shortcut to.  Instructions for how to do this can be found at [lifehacker](http://lifehacker.com/343328/create-a-keyboard-shortcut-for-any-menu-action-in-any-program). But it's a problem if the menu item changes dynamically.

<img src="dynamic-menu-item-shortcut-preference.png" alt=""/>

I’ve been using [Versions](http://versionsapp.com/) as a svn client for a couple of weeks to help manage my writing, and so far it’s proven to be a great app. But in managing LaTeX projects, there are a lot of files that should to be ignored by the repository. Unfortunately, Versions doesn’t have a keyboard shortcut for ignoring a selected file. To make matters worse, the name of the menu item changes to reflect the selected file, so making a keyboard shortcut in the usual way doesn’t work.

<img src="ignore-item-no-shortcut.png" width=""/>

I guess this wouldn’t be so bad if I weren’t a keyboard junkie. But, here’s how I worked around it… Open Automator, and choose the “Service” template for the new Automator action.

<img src="automator-service.png" alt=""/>

Tell the service to take no input, and to only work in the App you want to make a shortcut for.

<img src="service-receives.png" alt=""/>

Add an applescript block to the workflow. Here’s where we’re going to activate the menu item. In Versions, the menu item I wanted to access always begins with the word “Ignore”. For this particular trick to work, your target menu item needs to have at least one stable identifying string in its name. The code also needs to describe where in the menu hierarchy your menu-item of interest is found. Here’s the code for getting at the Ignore command in Versions:

```applescript
on run {input, parameters}
  tell application “System Events”
    tell process “Versions”
      set menuList to name of every menu item of menu “Action” of menu bar 1
      repeat with itemName in menuList
        if itemName contains “Ignore” then
          click menu item itemName of menu “Action” of menu bar 1
          return
        end if
      end repeat
    end tell
  end tell
end run
```

Upon saving, the workflow should save itself to <code>~/Library/Services</code> and should now be available as a service in your target application.

Now, back in the keyboard preferences, your new service should be available in the list of services, and you can assign whatever keyboard shortcut you like!

<img src="keyboard-shortcut-preferences.png" alt=""/>
