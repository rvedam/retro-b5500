# WebUI Using the Card Reader #



The B5500 supported several card reader models, with speeds ranging from 200 to 1400 cards per minute. The higher-speed models had a belt-driven feeding mechanism that was similar to the Burroughs check-sorting equipment, and was probably derived from it.

A B5500 system could have one or two card readers, identified as `CRA` and `CRB`.

Here is a view of either a B124 or B129 reader (they were physically identical, differing only in speed) at Stanford University in California, probably during the mid-1960s:

> ![https://googledrive.com/host/0BxqKm7v4xBswRjNYQnpqM0ItbkU/B5500-at-Stanford.jpg](https://googledrive.com/host/0BxqKm7v4xBswRjNYQnpqM0ItbkU/B5500-at-Stanford.jpg)


# Background #

The card reader interface we have developed for the web-based emulator is modeled after the 1400 card-per-minute B129. This interface opens in a separate window when the **POWER ON** button is activated on the emulator console:

> ![https://googledrive.com/host/0BxqKm7v4xBswRjNYQnpqM0ItbkU/B5500-CardReader.png](https://googledrive.com/host/0BxqKm7v4xBswRjNYQnpqM0ItbkU/B5500-CardReader.png)

The B129 had additional buttons and lamps related to the mechanical issues of reading cards (e.g., feed and read check indicators), but these controls are not relevant to operation under the emulator.

Card "decks" used with the emulator are ordinary ASCII text files, which can be prepared with any text editor. The files can have any file-name extension; we recommend using "`.card`". Lines of text in the file may be delimited with a carriage return (ASCII hex 0D), a line feed (ASCII hex 0A), or a carriage-return/line-feed pair.

When reading in alpha mode, each line of text represents one 80-column card image. Lines of text shorter than 80 characters will be padded with spaces internally on the right to a length of 80; lines longer than 80 characters will be truncated internally on the right to a length of 80.

When reading in binary mode, each line of text should have 160 characters, arranged in pairs. The first character of each pair represents the binary value of the top six rows in a column of the card; the second character represents the binary value of the lower six rows in that column. Each ASCII character from the binary card image is translated to its equivalent 6-bit B5500 internal code, e.g., "`A`" to 21 octal, "`$`" to 52 octal. Lines shorter than 160 characters will be padded internally on the right with zeroes to a length of 160; lines longer than 160 characters will be truncated internally on the right to a length of 160.

Binary cards are generally used only for small bootstrap-loader programs; they are not normally read by user programs.

In either mode, lines of text should be composed using the emulator's version of the B5500 64-character set:

```
    0 1 2 3 4 5 6 7
    8 9 # @ ? : > {
    + A B C D E F G
    H I . [ & ( < ~
    | J K L M N O P
    Q R $ * - 0 ; {
      / S T U V W X
    Y Z , % ! = } "
```

The B5500 used five special Algol characters that do not have ASCII equivalents. The emulator uses the following ASCII substitutions for them:

  * `~` for left-arrow
  * `|` for the multiplication sign
  * `{` for less-than-or-equal
  * `}` for greater-than-or-equal
  * `!` for not-equal

The card reader interface will translate lower-case ASCII letters to upper case. The underscore ("`_`") will be accepted as a substitute for `~` as the left-arrow. The reader will also accept five Unicode characters for the special Algol characters:

  * U+00D7: small-cross (multiplication sign)
  * U+2190: left-arrow
  * U+2260: not-equal
  * U+2264: less-than-or-equal
  * U+2265: greater-than-or-equal

The card reader will consider all other ASCII or Unicode characters to be "invalid punches" in a card. A "`?`" in the first position in a line of text will be considered to be an invalid punch (for the purpose of identifying the line to the MCP as a control card) but will be allowed as a valid character in any other position.

Your card-deck text files should normally have the necessary control cards on the front (with a "`?`" or other invalid character in the first column) and a `?END` control card at the end, but this is not a requirement of the emulated reader. A single text file can represent a complete deck, multiple decks, or a partial deck. While the physical card readers had an input hopper capacity of about 2400 cards, there is no practical limit to the size of a text file that can be loaded into the emulated reader.


# Card Reader Control Panel #

The user interface for the emulated card reader consists of the following controls and indicators:

  * **NOT READY** -- this white indicator illuminates when the reader is in a not-ready status. The reader becomes ready when the **START** button is clicked and the "input hopper" is not empty. It becomes not-ready when the **STOP** button is clicked or after the last card is read from the input hopper.
  * **EOF** -- This red button/indicator is used to signal end-of-file to the host system. When this button is activated (the indicator is lit), clicking the **START** button with an empty input hopper will set an EOF indication in the B5500 result descriptor. The MCP ignores this indication, however, so the button is non-functional with the B5500.
  * **STOP** -- clicking this red button will stop the reader and place it in a not-ready status. The **NOT READY** button will illuminate.
  * **START** -- clicking this green button when the input hopper is non-empty will place the reader in a ready status. The **NOT READY** lamp will go out. The MCP should sense the status change within a second or two and begin (or resume) reading cards.

Below the buttons is a file-picker control. Clicking its **Browse...** or **Choose File** button will open a dialog box from which you can select files to load as card decks into the reader. This control is enabled only when the reader is in a not-ready status. You can select multiple files at a time, which will cause all of the files selected to be appended to the input hopper. The order in which they are appended depends on your browser and underlying operating system, however.

You can append additional files to the reader's input hopper at any time. If the system is currently reading cards, simply click **STOP** to make the reader not-ready, select the files you want to load, and then click **START** to resume reading.

Below the file-picker control is a progress bar. This shows the relative size of the stack of cards remaining in the reader's input hopper. Each time you load one or more files into the reader, this bar advances to its full width. As cards are read, the length of the bar will decrease towards the left in proportion to the number of cards left to be read. See below on how the progress bar can be used to "empty the hopper."

Below the progress bar, the reader shows the last two card images that were read. The most recently-read card image is on the bottom. This can be useful if the MCP stops the reader due to a control-card error or an invalid character detected in a card.


# Operating the Card Reader #

The basic technique with the card reader is simple -- use the file-picker control to load one or more files into the reader, then click the **START** button to make the reader ready. The MCP should recognize the ready status and begin reading cards automatically. The reader will become not-ready after the last card has been read.

As mentioned above, you can stop the reader at any time and make it not-ready by clicking the **STOP** button. Restart it by clicking the **START** button. Whenever the reader is stopped and in a not-ready status, you can load additional files into it.

You can also "empty the hopper" while the card reader is in a not-ready status. To do this, click the progress bar. An alert box will pop up asking you to confirm that you want to empty the hopper.

Note that it is not possible to remove just one of your "decks" from the reader's input hopper. Once files are loaded into the reader, they lose their individual identity as files and become just a part of the stack of cards to be read. When you empty the hopper, everything goes, regardless of the file or files from which it originally came.

When the MCP encounters a control card with an error, or a card with an invalid character in other than the first column, it will stop reading cards and display the card in error on the SPO. The reader will remain in a ready status. When this happens, you generally have two choices:

  1. Stop the reader, empty the input hopper, correct the file with the card in error off-line, then reload the corrected file into the reader.
  1. On the SPO, enter "`CL CR`_x_" or "`RY CR`_x_" (where "_x_" indicates the reader, `A` or `B`) to clear the MCP's error status. The MCP will resume reading cards, but will not process them until after the next `?END` card is encountered. This is a convenient way to bypass just the deck that has an error.

The reader shows the last two cards that were read in the bottom panel of its window. This should help you locate the card in error so that you can correct it.

On some browsers, attempting to load the same file twice in succession into the reader may not work. The reason is that the emulator relies on the HTML file-picker control's "onChange" event to detect when you have selected a file. If you reselect the same file you just previously selected, the browser may not consider the value of the file-picker control to have changed, and hence not fire the event.

The browser resets the file-picker control whenever the reader goes not ready, either when the input hopper becomes empty or you click the **STOP** button. Thus, to load the same file multiple times in succession into the reader, simply click **STOP** between each file selection.