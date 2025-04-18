---
layout: page
title: Developer Guide
---
* Table of Contents
{:toc}

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

* {list here sources of all reused/adapted ideas, code, documentation, and third-party libraries -- include links to the original source as well}

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

<div markdown="span" class="alert alert-primary">

:bulb: **Tip:** The `.puml` files used to create diagrams in this document `docs/diagrams` folder. Refer to the [_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create and edit diagrams.
</div>

### Architecture

<img src="images/ArchitectureDiagram.png" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<img src="images/ArchitectureSequenceDiagram.png" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<img src="images/ComponentManagers.png" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

![Structure of the UI Component](images/UiClassDiagram.png)

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` and `Meeting` objects residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<img src="images/LogicClassDiagram.png" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

![Interactions Inside the Logic Component for the `delete 1` Command](images/DeleteSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</div>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a contact).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<img src="images/ParserClasses.png" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<img src="images/ModelClassDiagram.png" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` and `Meeting` objects (which are contained in a `UniquePersonList` and `UniqueMeetingList` object respectively).
* stores the currently 'selected' `Person` and `Meeting` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` and `ObservableList<Meeting>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<div markdown="span" class="alert alert-info">:information_source: **Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<img src="images/BetterModelClassDiagram.png" width="450" />

</div>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<img src="images/StorageClassDiagram.png" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedAddressBook#commit()` — Saves the current address book state in its history.
* `VersionedAddressBook#undo()` — Restores the previous address book state from its history.
* `VersionedAddressBook#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

![UndoRedoState0](images/UndoRedoState0.png)

Step 2. The user executes `delete 5` command to delete the 5th contact in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

![UndoRedoState1](images/UndoRedoState1.png)

Step 3. The user executes `add n/David …​` to add a new contact. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

![UndoRedoState2](images/UndoRedoState2.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</div>

Step 4. The user now decides that adding the contact was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

![UndoRedoState3](images/UndoRedoState3.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</div>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

![UndoSequenceDiagram](images/UndoSequenceDiagram-Logic.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</div>

Similarly, how an undo operation goes through the `Model` component is shown below:

![UndoSequenceDiagram](images/UndoSequenceDiagram-Model.png)

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</div>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

![UndoRedoState4](images/UndoRedoState4.png)

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

![UndoRedoState5](images/UndoRedoState5.png)

The following activity diagram summarizes what happens when a user executes a new command:

<img src="images/CommitActivityDiagram.png" width="250" />

#### Design considerations:

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire address book.
  * Pros: Easy to implement.
  * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
  * Pros: Will use less memory (e.g. for `delete`, just save the contact being deleted).
  * Cons: We must ensure that the implementation of each individual command are correct.

_{more aspects and alternatives to be added}_

### \[Proposed\] Data archiving

_{Explain here how the data archiving feature will be implemented}_


--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**:

* A Student that is looking for internships
* Has a need to manage a large number of contacts from their network
* prefer desktop apps over other types
* can type fast
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps

**Value proposition**: manage contacts in their network faster than a typical mouse/GUI driven app


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​         | I want to …​                                    | So that I can…​                                             |
|----------|-----------------|-------------------------------------------------|-------------------------------------------------------------|
| `* * *`  | First-Time User | view the available commands and their format    | learn how to send commands                                  |
| `* * *`  | User            | add a new contact                               | record a new contact in my network                          |
| `* * *`  | User            | delete a contact                                | remove a contact from my network                            |
| `* * *`  | User            | view all contacts                               | view all the contacts in my network                         |
| `* * *`  | User            | search for a specific contact by name           | look for a contact by their name                            |
| `* * *`  | User            | view an individual contact                      | view an individual contact in my network                    |
| `* * *`  | User            | sort my contacts by their importance            | view the contacts I'm most interested in contacting         |
| `* * *`  | User            | filter my contacts by hiring tag                | view my contacts currently looking for interns              |
| `* * *`  | User            | assign a hiring tag to a specific contact       | indicate whether they are open to hiring                    |
| `* * *`  | User            | view list of all meetings                       | keep track of all my meetings                               |
| `* * *`  | User            | create a meeting item                           | record down an upcoming/occured meeting                     |
| `* * *`  | User            | delete a meeting item                           | cancel a meeting                                            |
| `* * *`  | User            | update a meeting item                           | change details of a meeting                                 |
| `* *`    | User            | edit a contact                                  | edit the details of a contact in my network                 |
| `* *`    | User            | view number of contacts                         | view the size of my network                                 |
| `* *`    | User            | sort my contacts by name in alphabetical order  | view the contacts in my network in alphabetical order       |
| `* *`    | User            | sort my contacts by ascending/descending order  | view the contact list in reverse order                      |
| `* *`    | User            | search for a specific contact by company        | look for a contact by their company                         |
| `* *`    | User            | assign an importance tag for a specific contact | indicate their importance or relevance to me                |
| `* *`    | User            | record when my next meeting with them is        | keep track of future meeting date/time                      |
| `* *`    | User            | record where my next meeting with them is       | keep track of future meeting locations                      |
| `* *`    | User            | record notes about my meeting                   | keep track of what was mentioned during the meeting         |
| `* *`    | User            | view list of all previous meetings              | review all my past meetings                                 |
| `* *`    | User            | view list of all upcoming meetings              | keep track of any upcoming meetings                         |
| `* *`    | User            | add a contact to my meeting item                | keep track of who I meet during the meeting                 |
| `*`      | User            | sort my contacts by last meeting                | view contacts that I have recently met with                 |
| `*`      | User            | filter my contacts by their tags                | view contacts relevant to my current interest               |
| `*`      | User            | assign an industry tag to a specific contact    | create a custom tag with their industry                     |
| `*`      | User            | record when I last met my contact               | update the last-seen status if meeting with contact again   |
| `*`      | User            | record where I last met my contact              | update the last-seen location if meeting with contact again |

### Use cases

(For all use cases below, the **System** is the `AddressBook` and the **Actor** is the `user`, unless specified otherwise)

**Use case 1: Add a new contact**

**MSS**

1.  User enters the command to add a contact into the input field, along with all details following the specified command syntax
2.  AddressBook shows a success message containing the details of the added contact
    Use case ends.

**Extensions**

* 1a. The name of the contact provided is an exact (case-insensitive) duplicate of a name already in the contact list
   * 1a1. AddressBook shows an error message.
 
   Use case ends.

* 1b. The name, email and/or phone number of the contact provided is invalid
   * 1b. AddressBook shows an error message.
 
  Use case ends.


**Use case 2: Delete a contact**

**MSS**
1.  User enters the command to delete a contact into the input field, along with the index of the specified contact
2.  AddressBook shows a success message containing the name of the deleted contact

   Use case ends.

**Extensions**
* 1a. AddressBook detects invalid index number
    * 1a1. AddressBook shows an error message
      Use case ends.

* 1b. AddressBook detects index does not exist
    * 1b1. AddressBook shows an error message
      Use case ends.


**Use case 3: View all contacts**
1. User enters command to view all contacts
2. AddressBook shows all contacts that have been previously added by the user

    Use case ends.

**Extensions**
* 2. Number of contacts exceeds the maximum number of contacts to be displayed
   * 2a1. AddressBook uses pagination to display contacts by batches
     Use case ends.


**Use case 4: Search for Specific Contact by Name**

**MSS**
1. User requests to search for contact by name
2. AddressBook shows the results of the search

   Use case ends.


**Use case 5: Edit an existing contact**

**MSS**

1.  From the page where contacts are listed, user enters the command to edit a contact into the input field, along with the index of the contact they would like to edit and all details to be edited following the specified command syntax
2.  AddressBook shows a success message containing all details of the edited contact

    Use case ends.

**Extensions**

* 1a. The edited name, email and/or phone number provided is invalid
   * 1a1. AddressBook shows an error message.
   Use case ends.

* 1b. AddressBook detects invalid index number
    * 1b1. AddressBook shows an error message
      Use case ends.

* 1c. AddressBook detects index does not exist
    * 1c1. AddressBook shows an error message
      Use case ends.


**Use case 6: Add a meeting**

**MSS**

1.  User enters the command to add a meeting into the input field, along with all details following the specified command syntax
2.  AddressBook shows a success message containing the details of the added meeting

    Use case ends.

**Extensions**

* 1a. Any of the people's names listed do not exist in the AddressBook
   * 1a1. AddressBook shows an error message.
   Use case ends.

* 1b. The command is missing datetime or contact fields
   * 1b1. AddressBook shows an error message.
   Use case ends.


**Use case 7: Delete a meeting**

**MSS**
1.  User enters the command to delete a meeting into the input field, along with the index of the specified meeting
2.  AddressBook shows a success message containing the name of the deleted meeting
Use case ends.

**Use case 8: Add a new meeting**

**MSS**

1.  User enters the command to add a meeting into the input field, along with all details following the specified command syntax
2.  AddressBook shows a success message containing the details of the added meeting
   Use case ends.

**Extensions**
* 1a. AddressBook detects invalid index number
    * 1a1. AddressBook shows an error message
      Use case ends.

* 1b. AddressBook detects index does not exist
    * 1b1. AddressBook shows an error message
      Use case ends.


**Use case 9: List all meetings**

**MSS**

1. User enters command to view all contacts
2. AddressBook shows all contacts that have been previously added by the user

    Use case ends.


**Use case 10: List all meetings**

**MSS**

1.  User enters the command to list all meetings
2.  AddressBook shows a success message containing the details of the added meeting

    Use case ends.

**Use case 11: Edit a new meeting**

**MSS**

1.  User enters the command to edit a meeting into the input field, along with all details following the specified command syntax
2.  AddressBook shows a success message containing the details of the added meeting

    Use case ends.

**Extensions**

* 1a. Any of the people's names listed do not exist in the AddressBook
    * 1a1. AddressBook shows an error message.
      Use case ends.

* 1b. AddressBook detects invalid index number
    * 1b1. AddressBook shows an error message
      Use case ends.

* 1c. AddressBook detects index does not exist
    * 1c1. AddressBook shows an error message
      Use case ends.

**Use case 11: View help for commands**
1. User enters command or clicks the help tab to get help for command usage
2. AddressBook opens a seperate window to display a snippet of the user guide
    with the commands and their usages

    Use case ends.

### Non-Functional Requirements

1. Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2. Should be able to hold up to 1000 contacts without a noticeable sluggishness in performance for typical usage.
3. A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
4. After entering a command, a user should be able to see a visible response message within 1 second
5. Should return search results for a network size of up to 100,000 within 1 second
6. Startup time should not exceed 2 seconds on a typical modern system
7. The system should gracefully handle and log errors without crashing

### Glossary

* **Contact**: An individual or entity whose details are stored in the application
* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **Network**: Group of individuals connected through professional or personal relationships
* **Private contact detail**: A contact detail that is not meant to be shared with others
* **Meeting**: Any interaction (past or upcoming) between the user and the contact

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<div markdown="span" class="alert alert-info">:information_source: **Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</div>

### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

### Deleting a contact

1. Deleting a contact while all contacts are being shown

   1. Prerequisites: List all contacts using the `list` command. Multiple contacts in the list. Contact cannot have existing meetings.

   1. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No contact is deleted. Error details shown in the status message. Status bar remains the same.

   1. Test case: `delete 1` but contact at index 1 has existing meetings<br>
      Expected: No contact is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_
