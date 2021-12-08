# Day-Night-Cycle-Tutorial
Two scripts which will allow for day and night cycles at a rate specified by the user.

## 1) Creating a Scene

Simply add a **plane** and some 3D objects on top of it, whether they be cubes, spheres or anything else, it matters little as they are simply there to show us the movement of 
shadows and in turn the movement of the directional light.

At this stage, you could change the name of the directional light (which will be automatically present in any new scene) to **Sun**.

## 2) Creating Lighting Presets and a Lighting Manager

The Lighting Preset menu will hold the conditions for our presets, allowing us to edit and modify the colours of fog, ambient colour and directional colour.

The Lighting Manager will control the direction of our light.

To do so, we need to create two new scripts, one called **LightingPreset**, and the other called **LightingManager**.

After these scripts have been made, open the **LightingPreset** script in Visual Studio and set it to be a ScriptableObject rather than a MonoBehaviour. This will allow us to 
use the script as a means to store data to draw upon in between scenes.

Additionally, we will need to add **[System.Serializable]** above the class definition, as well as **[CreateAssetMenu(fileName = "Lighting Preset", menuName = 
"Scriptables/Lighting Preset", order = 1)]** which will create a file and menu within Unity's editor.

Next, we need to add gradient variables into the script. These will allow us to edit the ambient, fog and directional colours. Create public gradients referring to these lights
and you should have something similar to this;

    public Gradient ambientColour;
    public Gradient directionalColour;
    public Gradient fogColour;

Save and go into Unity, make an instance of the Light Preset menu by right clicking in the Asset folder and selecting it in the menu under Create>Scriptables>LightingPreset.
Rename the instance as **Lighting Presets**.

![Screenshot 2021-12-08 171857](https://user-images.githubusercontent.com/72862464/145253962-929d4935-5f98-4aa6-b65f-4ddb7b575b53.jpg)

In the image above, I have already chosen the colours within my gradients, and suggest you do the same.

## 3) The Lighting Manager

Within the **LightingManager** script, create private variables for a Light called directionalLight, a reference to the **LightingPreset**, as well as a variable to keep track
of the time so that it is visible within the editor. Serialize all these fields so that they are exposed within the editor.

    //references
    [SerializeField] private Light directionalLight;
    [SerializeField] private LightingPreset preset;
    //variables
    [SerializeField, Range(0, 24)] private float timeOfDay;
    
Additionally, add an ExecuteInEditMode attribute above the class definition to allow certain methods in the class to be able to execute when in the editor.

This can be used, for instance, to adjust the time of day within the editor to see the effects it has on the scene.

Next, we will write in an OnValidate function that will be called everytime the script is reloaded or something is changed within the inspector, and, unless specified otherwise,
will set the directional light as the one that is currently within the scene.


    {
        if (directionalLight != null)
        {
            return;
        }
        if (RenderSettings.sun!=null)
        {
            directionalLight = RenderSettings.sun;
        }
        else
        {
            Light[] lights = GameObject.FindObjectsOfType<Light>();
            foreach (Light light in lights)
            {
                if (light.type == LightType.Directional)
                {
                    directionalLight = light;
                    return;
                }
            }
        }
    }
  
  Next, we will need a component that will change the lighting settings depending on the time of day.
  
  We will use an input variable, ranging from 0 to 1, and then set the RenderSettings to evaluate the gradients in the lighting preset dependant on the time of day.
  
  Additionally, we will need to ensure that we have actually assigned a directional light, then set the colour and the rotations depending on the time of day using a 
  loop function. Create a private void UpdateLighting(float timePercent) and within the function write this:
  
    {
        RenderSettings.ambientLight = preset.ambientColour.Evaluate(timePercent);
        RenderSettings.fogColor = preset.fogColour.Evaluate(timePercent);

        if (directionalLight!=null)
        {
            directionalLight.color = preset.directionalColour.Evaluate(timePercent);
            directionalLight.transform.localRotation = Quaternion.Euler(new Vector3((timePercent * 360f) - 90f, 170f, 0));
        }
    }
  
  Finally, we will write an Update method to ensure the lighting is changing frame by frame.
  
  First, we will want to check that we have assigned a lighting preset through an if statement in the Update function.
  
    {
        if (preset==null)
        {
            return;
        }
  
  Next we will want to update the lighting along with the time as the application is playing, which will also be done in the Update function.
  
        if (Application.isPlaying)
        {
            timeOfDay += Time.deltaTime * 0.5f;
            timeOfDay %= 24; //clamp between 0-24
            UpdateLighting(timeOfDay / 24f);
        }
        else
        {
            UpdateLighting(timeOfDay / 24f);
        }
    }
  
  Additionally, by applying different multiplications to Time.deltaTime, we can adjust the length of the day and night cycle.
  
  ## 4) Testing the Cycle in Unity
  
  Now, add an empty game object into the scene and call it something along the lines of **Lighting Manager** and attach the **LightingManager** script to it. 
  Now, you should be able to see in the editor the lighting manager script functions.
  In the editor, assign the Lighting Preset asset to the variable, so the menu in the editor should looks like this:
    
![Screenshot 2021-12-08 171955](https://user-images.githubusercontent.com/72862464/145253993-3bf68b68-8a22-422a-814e-978bd0c37981.jpg)

  Now, when moving the slider for the time of day, the directional light should be moving in the scene, casting light from different angles.
  
  When playing the application, the day and night cycle will begin, with the time of day increasing by itself while the lighting will update after.
