# mobile-a11y-knowledge
Some knowledge for mobile a11y

## glossary

A11y order (alternative to A11y sequence): the traverse order child elements of the screen (element or component)

## iOS Voice Over Order

### User cannot direct select a button(one finger single tap) in VoiceOver mode if the button is visually contained by (smaller and inside of ) higher priority buttons.

#### Situation
In a video screen that is fullscreen, there were a close button on top left, play/pause button which has frame = fullscreen, then artist button, follow button... There is a problem that the play/pause is fullscreen thus when I directly select (one finger single tap) follow button, it focus on play/pause.

#### Finding
I can directly selected (one finger single tap) close button not any other elements. 

#### Guess 
It has to do with the accessibility sequence.

#### Experiment
moving playPause after caption, followView, actionButtonRow

```swift
// original
  accessibilityElements = [closeButton, playPause, caption, followView, actionButtonRow]

// moved playPause to the end
  accessibilityElements = [closeButton, caption, followView, actionButtonRow, playPause] 
```

#### Result
Applied this change, I am able to direct select all buttons on screen. However the playPause goes after other elements, where does not meet task expectation.

#### Failed attempts
1. z-index, does not affect sequence or playpause blocking later elements


### Override tap action of an UI in swift

``` swift
    @objc func handleTap() {
        // expected action here
    }
    ...

    let tapGesture = UITapGestureRecognizer(target: self, action: #selector(handleTap))
    target_button.addGestureRecognizer(tapGesture) // overwrite tap action
```
