# How To: Modding Eco
A short tutorial based on modding the game Eco by Strange Loop Games
## Getting Setup
### Visual Studio
**NOTE:** Versions of Visual Studio and .NET Framework change over time verify what version you need.
- Download Visual Studio 2022 from https://visualstudio.microsoft.com/downloads/
- Download .net framework 7.x from https://dotnet.microsoft.com/en-us/download/dotnet/7.0
- Install the framework and Visual Studio installer.
- When the installer opens select the following options:
	- ASP.NET and web development.
	- .NET desktop development.
	- Desktop development with C++ (select latest windows SDK)
	- Game development with Unity (deselect unity editor)
- Now open and hit create a new project.
- Select class library.
- Name your project and solution.
- select.NET Framework 7.0 if you dont see it you need to install from step 2.
- Hit create.
- Right click the project for the project and go to manage nuget packages, select the browse tab and check include pre-releases then search for `Eco.ReferenceAssemblies` and install it.
- Start coding!
#### Optional Formating
- Right click the project in solution explorer.
- Click "properties"
- Scroll down to default namespace and set it to `Eco.*Company Name*`
- Make folders for the types of classes the mod has like:
	- Items for Items
	- Objects for WorldObjects
	- Components for WorldObjectComponents
	- Tooltips for TooltipLibrarys
	- Util for any utilities like world object functions
- If a folder gets crowded make sub folders but not new namespaces
### Unity
- Download and install Unity Hub from https://unity.com/download
- Download the modkit from https://play.eco/account
- Unzip the modkit into any folder.
- Download the Unity version listed in the `ProjectVersion.txt` file.
- Create a new 3d project.
- Delete the sample scene and import the `EcoModKit.unitypackage` from the mod kit.
	- Right click assets.
	- Go to `import package`.
	- Click `custom package`.
	- Select the file from the modkit folder.
	- Hit import.
- Start designing!
#### Optional Formating
- Make a folder in the root of the assets with your company name.
- Make two folders named `Template` and `Mods` within that folder.
- inside the template folder make a Scene named `Eco.*Company Name*.Template`.
- them make a `Prefabs` and `Assets` folders.
- paste the ItemTemplate from `Assets>EcoModKit>Prefabs>ItemTemplate` and paste it into `Assets>*Company Name*>Template>Prefabs>Templates`.
- open the `Assets>*Company Name*>Template>Eco.*Company Name*.Template` scene.
- make 4 empty game objects.
- name the first one `Items`.
- name the second one `Objects` and add the `Modkit Prefab Container` component to it.
- name the third one `Emoji` and add the `Chat Emote Set Old` component to it.
- name the fourth one `BlockSets` and add the `Block Set Container` component to it and set the list size to zero.
- save the scene.
- make an empty game object and name it `WorldObject`.
- add a `World Object` component to it and set its layer to `ModObject`.
- Save the object as a prefab in `Assets>*Company Name*>Template>Prefabs>Templates`.
- delete the object from the scene.
- save the scene again just in case.
#### Cloning the template
- ensure you have done the "Optional Formating" section or skip this.
- copy the `Assets>*Company Name*>Template` folder to `Assets>*Company Name*>Mods`.
- rename the new folder to `Eco.*Company Name*.*Mod Name*`.
- rename the scene inside within the new folder to the same.
- begin!
## Making a basic Item
### Server Side
- First make a new file called `*Item Name*.cs` in the Items directory.
- Add a partial class called `*Item Name*Item` that extends the `Item` class.
- All Items and WorldObjects need a `Serialized` attribute dont forget to add it unless you like no nboot servers.
- Add Attributes to the item as needed.
- some `ItemAttributes` include:
	- Functional
		- `WaterPlaceable`: Can be placed on water.
		- `Fuel`: states the fuel value of an item.
		- `Carried`: makes the item only carryable.
		- `Weight`: sets the weight of that item x1.
		- `MaxStackSize`: sets the max items in a stack.
		- `RequiresTool`: Item can only be carried with this tool.
	- Tooltip related:
		- `LocDisplayname`: Sets the localized name shown ingame for the item.
		- `LocDescription`: sets the localized description of the item ingame.
		- `AirPollution`: Shows the PPM of polution the Object linked to the item makes.
		- `Tier`: Shows the material Teir.
		- `LiquidProducer`: Shows a produced liquid and the rate.
### Item Code Example
```cs
[Serialized]
[Tag("Battery")]
[Weight(20), MaxStackSize(1)]
[LocDisplayName("Small Battery"), LocDescription("A device for storing power for later use.")]
public partial class SmallBatteryItem : Item { }
```
### Client Side
- setup your mod folder.
- copy the `ItemTemplate` prefab into the `Items` game object.
- right click the prefab and select prefab then unpack completely.
- rename the cloned object to the name used on the server so if the class is named "PotatoItem" name the object "PotatoItem".
- import the image asset to unity and put it in your mod folder under assets (organization is up to you)
- select the forground object (its a child of the ItemTemplate)
- look for the `Image` component in the inspector window.
- look for `Source Image` and set it to the sprite you imported earlier.
- congrats you now have a item icon!
### Giving the item a recipe
- In the file you made the item in add a new partial class called `*Item Name*Recipe` extending `RecipeFamily`
- Add a constructor and set Recipes to a list with recipes inside.
- The `Recipe` class has a few arguments for the constructor:
	- `name`: a string having a unique recipe name.
	- `displayName`: a LocString having the name but doesnt need to be unique.
	- `inputs`: a list containing `IngredientElement`s which each take an item type and a quantity and an optional argument of if its a static cost.
	- `outputs`: a list containing `CraftingElement`s which take a type argument of the item type its producing then an argument of quantity.
- Set `ExperienceOnCraft` to the base experience gained.
- Set `LaborInCalories` to the output of `CreateLaborInCaloriesValue` which takes arguments for base labor and skill type.
- Set `CreateCraftTimeValue` to the output of `CreateCraftTimeValue` which takes arguments for the type of this RecipeFamily, craft time in minutes, type of the skill used, list of talents that can be utilized (ie. `ParallelProcessingTalent` and `FocusedWorkflowTalent`)
- Initialize the class using `Initialize` which takes arguments of a LocString for the display name and the type of this RecipeFamily.
- Add the recipe to a work table using `CrafingComponent.AddRecipe` which takes arguments of WorldObjectType and the RecipeFamily to add (just put `this`)
- Add any needed attributes.
- Some notible attributes are:
	- `RequiresSkill`: shows the required skill in the tooltip.
	- `RequiresModule`: Requires another object in the same room.
### Recipe Example:
```cs
[RequiresSkill(typeof(SmithSkill), 1)]
[RequiresModule(typeof(CampfireObject))]
public partial class SmallBatteryRecipe : RecipeFamily
{
    public SmallBatteryRecipe()
    {
        Recipes =
        [
            new Recipe(
                "Small Battery",
                Localizer.DoStr("Small Coal Battery"),
                [
                    new IngredientElement(typeof(IronBarItem), 8, true),
                    new IngredientElement(typeof(CopperBarItem), 8, true),
                    new IngredientElement(typeof(CoalItem), 24, false),
                ],
                [
                    new CraftingElement<SmallBatteryItem>(1),
                    new CraftingElement<TailingsItem>(1),
                ])
        ];
        ExperienceOnCraft = 5;
        LaborInCalories = CreateLaborInCaloriesValue(500, typeof(SmithSkill));
        CraftMinutes = CreateCraftTimeValue(typeof(SmallBatteryRecipe), 5, typeof(SmithSkill), [typeof(FocusedWorkflowTalent), typeof(ParallelProcessingTalent)]);
        Initialize(Localizer.DoStr("Small Coal Battery"), typeof(SmallBatteryRecipe));
        CraftingComponent.AddRecipe(typeof(AnvilObject), this);
    }
}
```
## Making World Objects
### Visual Studio
- Make a new partial class called `*Item Name*Object` that extends the `WorldObject` class and implements the `IRepresentsItem` interface.
- inside the class make a public Type property called `RepresentedItemType` that is assigned to the typeof the item that places this object.
- make a new partial class called `*Item Name*Item` that extends the `WorldObjectItem<*Item Name*Object>`.
- Add any attributes and ensure both classes have the `Serialized` attribute.
- ### Placeable Item Code Example
```cs
[Serialized]
[LocDisplayName("Battery Charger"), LocDescription("Charges batteries using power provided by the grid.")]
public partial class BatteryChargerItem : WorldObjectItem<BatteryChargerObject> { }

[Serialized]
[LocDisplayName("Battery Charger")]
public partial class BatteryChargerObject : WorldObject, IRepresentsItem
{
    public Type RepresentedItemType => typeof(BatteryChargerItem);
}
```
### Unity
- Clone the "WorldObject" prefab you made earlier then unpack it completly and name it the same name as the object class just like items.
- Insert the Models as children to the game object.
- ensure that all materials used are using the `curved/standard` shader.
- drag the game object into the `mod folder>Prefabs` somewhere.
- select the object named `objects` in your scene and go to the inspector tab and expand the list of prefabs.
- drag the WorldObject prefab with your model into a new element on the list.
## Building the mod
- Build the visual studio code and find the .dll file from the project.
- make a folder named `Mods` then inside make a folder with your company name then a folder for each mod called there mods name.
- insde each mod folder paste the .dll file for it and make a folder called `Assets`.
- in unity go to `ModKit>Build current bundel` in the action bar.
- select the assets folder as the build location and build.
- once all the mods are built zip the mods folder and deploy!
- if you need to test (which you will) just use symlinks to link the dlls to the build folder and the build folder to your sever mods folder.
