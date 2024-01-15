# Using Image Builder

<procedure title="Adding a new application">

<step>
Launch an Image Builder instance from the AppStream 2.0 console using an existing base image or a shared image.
</step>
<step>
    Connect to the Image Builder instance using the streaming URL provided in the console. You can connect via the AppStream 2.0 client or a web browser.
</step>
<step>
    Install and configure your applications on the Image Builder just as you would on a regular desktop. Test that your applications work as intended.
</step>
<step>
    <p>Use the AppStreamImageBuilder tool on the desktop .</p>
     <list>
     <li>
         Choose a name for the application.
     </li>
     <li>
         Under launch path, enter the location of the exe used to launch the application.
     </li>
     <li>
            If any launch parameters are required, enter them under launch parameters.
     </li>
     </list>
    <img src="image-builder-setup-01.png" alt=""/>
    <list>
    <li>
        Click Next.
    </li>
    <li>
        Under test, run as template user.
    </li>
    <li>
        This will launch the template user, from here open the application via the AppStreamImageBuilder tool on the Desktop.
    </li>
       <li>
            Once the application is open, set any defaults required for the application.
       </li>
</list>

<img src="image-builder-setup-02.png" alt=""/>
</step>
<step>
    From the Image Assistant, select run as test user to test the image. This will launch a new fleet using the image you created. You can connect to the fleet using the streaming URL provided in the console. You can connect via the AppStream 2.0 client or a web browser.
</step>
<step>
The new image will appear in the image registry section of the AppStream 2.0 console with a status of "Available" once created.
</step>
<step>
You can now launch new fleets using your custom image which will have your applications pre-installed for users.
</step>
</procedure>

<procedure title="Optimise an Application">
<step>
<p>Select administrator user</p>
<img src="image_builder_template_user.png" alt=""/>
</step>
<step>
<p>Open image assistant</p>
</step>
<step>
<p>Go to step 4</p>
<img src="image_builder_optimise_step_4.png" alt=""/>
</step>
<step>
Follow the on-screen instructions.
</step>
</procedure>

<procedure title="Enabling Windows Anti Virus">
<warning>Antivirus adds overhead to virtualised instances, making it is a best practice to mitigate unnecessary activities. For example, scanning the system volume (which is ephemeral) at boot, for instance, does not add to the overall security of AppStream 2.0. </warning>
</procedure>
