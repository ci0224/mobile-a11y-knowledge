# mobile-a11y-knowledge
Some knowledge for mobile a11y

## glossary

A11y order (alternative to A11y sequence): the traverse order child elements of the screen (element or component)

## iOS Voice Over Order

### Overlapping elements can negatively impact accessibility. 

One sentense: If overlap buttons serve the same purpose, consider merging them; otherwise, keep them not overlapped.

User cannot direct select a button(one finger single tap) in VoiceOver mode if the button is visually contained by (smaller and inside of ) higher priority buttons.


- Situation
   - In a video screen that is fullscreen, there were a close button on top left, play/pause button which has frame = fullscreen, then artist button, follow button... There is a problem that the play/pause is fullscreen thus when I directly select (one finger single tap) follow button, it focus on play/pause.
- Finding
   - I can directly selected (one finger single tap) close button not any other elements. 
- Guess 
   - It has to do with the accessibility sequence.
- Experiment
   - moving playPause after caption, followView, actionButtonRow

```swift
// original
  accessibilityElements = [closeButton, playPause, caption, followView, actionButtonRow]

// moved playPause to the end
  accessibilityElements = [closeButton, caption, followView, actionButtonRow, playPause] 
```

- Result
   - Applied this change, I am able to direct select all buttons on screen. However the playPause goes after other elements, where does not meet task expectation.
- Failed attempts
   - z-index, does not affect sequence or playpause blocking later elements


### Override tap action of an UI in swift

``` swift
    @objc func handleTap() {
        // expected action here
    }
    ...

    let tapGesture = UITapGestureRecognizer(target: self, action: #selector(handleTap))
    target_button.addGestureRecognizer(tapGesture) // overwrite tap action
```

## Android Override Element Type

Type override will allow TalkBack to read the element type correspondingly when appropriate. 

> For example we might have a nice view component, which has onTap, containing many element, but system does not know this is a button.
> And a type override of Button will allow the TalkBack to recognize this is a Button.

```kotlin
ViewCompat.setAccessibilityDelegate(elementToBeOverridden, object : AccessibilityDelegateCompat() {
     override fun onInitializeAccessibilityNodeInfo(
         host: View,
         info: AccessibilityNodeInfoCompat
     ) {
         super.onInitializeAccessibilityNodeInfo(host, info)
         info.className = Button::class.java.name // Mimic a Button class
     }
 })
```

## Android accessibility / keyboard mode detection

```kotlin
    fun isAccessibilityEnabled(): Boolean {
        val am = context?.getSystemService(Context.ACCESSIBILITY_SERVICE) as AccessibilityManager
        return am.isEnabled && am.isTouchExplorationEnabled
    }

    fun isExternalKeyboardConnected(): Boolean {
        val inputManager = context?.getSystemService(Context.INPUT_SERVICE) as InputManager
        for (id in inputManager.inputDeviceIds) {
            val device = inputManager.getInputDevice(id)
            if (device != null &&
                !device.isVirtual && // This function will always return true, if virtual is accepted.
                (device.sources and InputDevice.SOURCE_KEYBOARD) == InputDevice.SOURCE_KEYBOARD &&
                device.keyboardType == InputDevice.KEYBOARD_TYPE_ALPHABETIC) {
                return true
            }
        }
        return false
    }
```

Known issue with isExternalKeyboardConnected():

1. this function will return false on virtual external keyboard(not too bad), for example connect Android device to laptop using [scrcpy](https://github.com/Genymobile/scrcpy) and using keyboard of laptop to control the Android devic.
2. If we remove the condition check of virtual, ` !device.isVirtual `, this funciton will always return true(very bad!).
