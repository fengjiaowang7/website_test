---
title: iOS Color Picker
meta:
- name: "twitter:card"
  content: "summary"
- name: "twitter:site"
  content: "@fcanas"
- name: "twitter:creator"
  content: "@fcanas"
- name: "og:title"
  content: "iOS Color Picker"
- name: "og:type"
  content: website
- name: "og:description"
  content: "A reusable color picker component for iOS. Works for iPhone, iPad, in modal sheets, popovers... just about anywhere."
---

A reusable color picker component for iOS. Works for iPhone, iPad, in modal sheets, popovers... just about anywhere.

# Using iOS-Color-Picker

## Installation

The easiest way to use iOS-Color-Picker is with [CocoaPods][0]. Add the following line to your `Podfile`.

```ruby
pod 'iOS-Color-Picker'
```

## Using a Color Picker from a View Controller

Suppose you have a view controller with a `color` property you'd like to let the user pick. Make your view controller implement the `FCColorPickerViewControllerDelegate` protocol. Handle the color picked and the cancelled methods, and make a method that triggers showing the view controller.

```objectivec
-(IBAction)chooseColor:(id)sender {
  FCColorPickerViewController *colorPicker = [FCColorPickerViewController colorPicker];
  colorPicker.color = self.color;
  colorPicker.delegate = self;

  [colorPicker setModalPresentationStyle:UIModalPresentationFormSheet];
  [self presentViewController:colorPicker animated:YES completion:nil];
}

#pragma mark - FCColorPickerViewControllerDelegate Methods

-(void)colorPickerViewController:(FCColorPickerViewController *)colorPicker didSelectColor:(UIColor *)color {
  self.color = color;
  [self dismissViewControllerAnimated:YES completion:nil];
}

-(void)colorPickerViewControllerDidCancel:(FCColorPickerViewController *)colorPicker {
  [self dismissViewControllerAnimated:YES completion:nil];
}
```

# Example Project

[iOS Color Picker Example][1]

[0]: http://cocoapods.org
[1]: https://github.com/fcanas/ios-color-picker-example
