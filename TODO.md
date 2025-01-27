# TODOs

- Fix scrolling in the log widget

- Rename `resizable` back to `shrinkable` or so.

- See src/widget/menu.cr. Move the <<, >> logic into a generic mixin

- All fibers and/or listeners must be recorded in respective classes so that they can be managed (removed/paused/detached etc.)

- In src/namespace.cr there is: `property label : Style { Style.new }`. Redesign that. Determine what to do with label.side. Possibly redo the whole label thing.

- In Blessed's version of examples/hello, it is not necessary to manually #clearPos(). Where does the difference compared to Crysterm come from?

- Maybe add a GUI-dedicated thread like in Qt?

- Do code2attr / attr2code legitimately belong to Screen, or they're better suited for some other file/place?

- When `Display` is created, if `TERM` env var is not defined it defaults to `xterm` or `windows-ansi`. Make this more robust to also include the existing check for which terminal emulator is in use, and then use the default which matches the default term setting of that emulator.

- Overflow is currently property of screen and widget. See what the relation is and whether it all works correctly.

- For `OptimizationFlag`s listed in src/namespace.cr, make a list of all common terminal emulators and see which ones support which optimizations. Than make default optimizations turn on/off based on that (unless overriden by user).
`OptimizationFlag`s are set on a `Screen`.

- In src/screen.cr, some stuff is done in initialize, while it seems like enter/leave would be the correct places.

- Determine what is the exact current situation re. whether borders/angles can be drawn using ACS chars or Unicode chars? Is both supported or currently the code only does one?

- Same as Widget#hidden, rename Cursor#_hidden

- Make cursor be property on widget. Now it's on screen. Or, just allow widget-specific one to override default one.

- Make Widget#content be original, user-supplied content. Name all other accessors differently (same type of solution as for left/right/etc.)

- If the screen is too small to display a widget in layout, don't hide it completely, make sure that at least something is drawn even if incomplete or incorrect. See e.g. small-tests/pine.cr for an example

- When aligning widgets, see if it is possible to control what char will fill the empty space, instead of always ' '

- Verify color names specifically listed in tput's `src/tput/output/text.cr` and check if there are any discrepancies compared to color names or name syntax and pattern listed/supported in the `term_colors` shard. If yes, make them uniform.

- Make uniform passing of args to classes, with class decls, function arguments, and arg.try... (E.g. disable unnamed parameters to all methods that have non-trivial args)

- It is not defined what happens if a Widget has parse_tags true, and there is syntax error in the tags. A syntax error is something as simple or just { or }. In the case of one {, { remains in input and the rest is removed. In all other cases (more {s or one or more }s), the whole section is removed.  This should be fixed/standardized. Offhand, either always removing everything, or always removing anything that's invalid as-is. (Didn't check yet how blessed does it.)

- Currently, default events in widgets are implemented in instance vars, and then when we want to enable/disable widget events, we either add or remove those handlers/vars from the events' handlers hashes. But the code for that is tedious/almost manual. Maybe all events should be in an array or something, and then adding or removing is just handlers.clear or handlers.push *array.

- Evaluate when events are triggered and how they are named. Events that are triggered before the code is executed should be named like e.g. 'Attach'. Events that trigger after the work has been done should be in past tense, e.g. 'Attached'.

- Style setting for determining whether cursor is visible in a widget or not. E.g. it should be possible to have cursor optionally appear in focused checkboxes and radiobuttons.

- Implement artificial cursor (with cursorFlashTime option). See how it's done in Blessed.

- See that whatever widgets have done on initialize are undo-ed when they or Screen they were on are destroyed

- Support "reverse" style in Style. There should be code which gives the "opposite" of any color. Then in code, when we detect overlapping colors which are too similar, one can simply be switched to its opposite. Minimal/beginning for this might be in existence. Search for "invert" and "attr".

- Support starting the app without switching to alt buffer and re-setting everything. This would make it more usable for line-oriented apps

- More widgets - from Blessed and `slap` text editor which is based on Blessed

- Add max len to text widgets

- Profile the app: `build --debug; perf record --call-graph dwarf ./app; hotspot perf.data`

- In Crysterm, which should aim to be fully OO & clean code, strings have been replaced with enum values in many places. However, when specifying widget sizes or position, percentages are still typically given with strings, e.g. "80%", top/left support a special string value of "center", and width/height support "resizable". Strings can stay as a supported option (for convenience of loading settings from text files, etc.?), but see if those can be replaced with enums or similar, and with things like 80.percent() or something.

- For good OO, and like in Qt, it would be good if all functions that deal with Points, Sizes, and Dimensions, would also accept those specific classes/structs that we'd define, rather than just numbers/Ints. This already exists to an extent, e.g. in Tput there is class Size, Point, etc. These should be used more throughout the codebase, and any other relevant new ones added.

- Performance improvements - can something substantial be done? In widgets and everywhere, but specifically:
(1) for draw() - would it help if there was a region to draw manually managed, or current do-all code is fine?
(2) for render/draw - any benefit from draw() being separately schedulable? Also, how to keep track of rendering and skip it if nothing has changed? And where in memory is rendering? in screen or in widget?
(3) Can parse_content be called less times, and can `if @parse_tags` be checked less times?

- Implement generic functions for all size/position values. Right now, top/left/width/height etc. can take various specifications, including "center", "resizable", "80%" etc. It would be good to completely streamline this, so that value can be set using all these options, but when reading it (e.g. through special getters) they would always return an int value if possible, and nothing else.

- Currently, one can set widget's content with "content: ...". However, the code manipulates this value on its own. And therefore text widgets have a custom "text" where the raw/user value is. See if it's possible that @content is never re-set to parsed/internal value, and that it always contains direct/raw user input. If possible, then @text hacks wouldn't be needed.

- When drawing to screen, @ret variable can be used for diverting output temporarily. See if, instead of that method which was inherited from blessed, it could be a block that yields and writes to given IO.

- Check if there if performance benefit to manually checking if there are event listeners before emitting events. We currently do this, but if the speed is the same, then we shouldn't bother checking in advance, but we should simply emit always.

- When specifying top/left, it is possible to say "center". This will center the widget, and is different than saying "50%". For example, for a screen of 100 and width 50, "center" will make it begin at 25, while "50%" will make it begin at 50. Now, when using percentages, it is possible to say e.g. "50%+10" (This would result in widget starting at 60 in our example). But it appears it is not possible to use the +- specifier if "center" is used. Support specifying +- in all cases.

- Currently, when a Screen is listening for keyboard, it does this keypress by keypress. See how this affects pasting blocks of text into the terminal via mouse. Maybe they could be a "batch" mode where, if a paste is detected (possibly by realizing that multiple bytes became available at the same time?), all this text is processed as a plain text, at once? Maybe with a flag/toggle to do so?

- Examine effect of `use_buffer` variable in Tput, and see whether it can be completely removed, or it can be used meaningfully in some way?

- See if it would be of any benefit to mark certain methods with @[AlwaysInline].

- In rendering, there is Overflow enum used as a return type, which defines what to do if a widget can't be rendered without overflowing. Add MoveWidget or similar as another option. It would have the effect of moving the widget so it can render. A use case for this would be e.g. auto-completion boxes or similar which pop-up. For simplicity the developer would just have them pop up at the desired location, and Crysterm would adjust for overflow automatically.

- When a Display starts listening for keys (possibly other stuff too), there is no way to cancel it, since there is no API to kill a Fiber from the outside. So once this is started, it's active til the program exits.
Not a huge deal since a Display unconditionally starts listening and emitting received stuff, but it's something to improve long term (there should be a way to gracefully stop listening and/or destroy/re-create the Display object).

- See how src/widget_children.cr and src/mixin/children.cr could be more integrated and how Screen->Widgets could re-use as much of it as possible

- Why is there a newline difference in output of blessed and crysterm's Screen#screenshot?

- Crystem resets terminal to cooked mode upon exit. But it should just return it to the state it was in before starting the app. Tput shard already has support for this, but it never activates. Look into the code, make tput's native functionality work, and then remove the manual resetting done in crysterm.

## Widget Fixes

Listed here since generic fixes/improvements have priority over widget-specific ones:

- Layout widget has a calculation error in masonry style. It's present in blessed and it got carried over here. See how this could be fixed.

- Separate Layout widget's two possible layouts into separate widgets.

- In small-tests/question.cr, see if the widget can be fixed to work properly, or it's not worth it (since the original implementation of the widget in Blessed is quite weird, maybe it should be redone)

- Fix for TextArea's _done; make sure that both examples/hello2.cr and prompt/question example work

- Make sure that chars typed in text input are immediately rendered (i.e. not holding 1 in buffer in non-release mode). Hopefully the only issue here is just timing, i.e. the way how render() call schedules a render.

- Widget::Prompt - determine the reason for 1 cell difference in positioning of "Question" and "Cancel". Does it have to do with auto_padding which is now default? If yes, just ignore this issue.

- Crysterm is missing a full, 100% working TextArea widget (same as Blessed). TextArea that exists is very basic. Dbkaplun wrote Slap, text editor based on blessed. Try to port its text editor widget to Crysterm

- Add the top of `def _render`, there is code added checking for _is_list etc., to be able to style list items correctly. But, this probably needs to go into a render() function which is subclassed in List, rather than being present in global _render

- In Blessed, there are a couple _isX variables. They can be replaced with #is_a?(Class) in Crysterm, and that has been done in some places. But, this might make users unable to set _is_x in their own widgets, if they don't want to inherit from List etc. See if _is_x are actually better than is_a?s. Or figure out how someone would get e.g. _is_list behavior from Crysterm's core without inheriting from List.

- In TextArea widget, arrows can be used for scrolling and it works correctly (even though the behavior is a little bit unintuitive). All this is inherited from Blessed. The unintuitive part is that the cursor position isn't reflected when using the arrows, so it's not clear that scrolling will eventually happen. Also positioning the cursor within existing text doesn't work because cursor can't move. An attempt to make it move just as a scrolling indicator was made, and it almost worked (see `to_scroll_pos` in TextArea). But the scroll pos is then hidden/reverted by code in Widget which calls _update_cursor() on widget that is in focus. See if this can be fixed to really work.

- src/widget/overlayimage.cr -> is that OK or more work needs to be done?

- src/widget/question.cr -> needs more work (check why it breaks with padding: 1)

## Misc

- Actions can theoretically be rendered in various forms, as e.g. checkboxes, menu entries, etc. See how to combine this with widget functionality. How does Qt do it?

- Why Tput has Cursor class, but no field for cursor color?
