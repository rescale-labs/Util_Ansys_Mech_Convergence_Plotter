# Ansys Mechanical Force Convergence Plots
![image](https://github.com/rescale/soln-wee-scripts/assets/64439634/2d99c234-fdcb-46ef-9a56-8ac6e87dc770)

When conducting a Non-Linear Ansys Mechanical simulation, the solver generates Force Convergence Plots that can be visualized within the Solution Information Branch of the Ansys Mechanical Tree. These graphical representations offer valuable insights into the progression of the solution. Specifically, the upper graph depicts the Model's Force Convergence plotted against the number of iterations performed, while the lower graph illustrates Simulation Time plotted against Cumulative Iterations.

Additionally, the upper graph highlights significant events such as Sub-Step/Load-Step convergence and Bisections.

However, when utilizing the Rescale platform for solving an Ansys Mechanical Job, access to these informative graphs is restricted. While it's possible to track solution progress in real-time through the progress_output.log (solve.out) file, this approach often falls short in providing a comprehensive analysis of the solution's temporal evolution.

To address this limitation, we offer the convergence.py script presented here. This script enables the recreation of Force-Convergence plots, empowering users to effectively monitor and assess the status of ongoing or completed Non-Linear Ansys Mechanical simulations.

# Working 

The script uses the python libraries pandas and matplotlib to regenerate the Force Convergence plots during runtime.

1. It reads the file.gst that contains the solution progression information. This file is created by the non-linear Ansys Mechanical solver.
2. It creates a cleaned version of file.gst after removing the Header data and names it cleaned_file.gst.
3. Data from the cleaned file is read into a Pandas Dataframe and the plot is generated using Python matplot libary.
4. The Force Convergence plot named convergence.png is then generated under the work/shared folder.

Users can then search for convergence.png from the livetailing menu to review the Force onvergence Graph while the job is running.

# Usage

Two operational modes are outlined below: the first automatically generates and updates the convergence.png plot while the Ansys Mechanical solver is running, and the second allows for manual generation of the plot from a Jupyter notebook session.

## Auto Generation During Runtime

1. Create a template:

For the automated Force-Convergence generation approach, it is advisable to create and save a template with the specified settings. This ensures that the template can be reused without manual command editing for each run.

2. Upload convergence.py as Template Input:

Include the convergence.py Python script as part of the template. This script will be reused for all runs.

3. Software Commands:

After selecting the desired Ansys Mechanical Version, prepend the following commands to install and configure a virtual Python environment with the required pandas and matplotlib libraries.

		pip3 install --user virtualenv
		python3 -m virtualenv $HOME/ansys
		source $HOME/ansys/bin/activate
		pip install matplotlib==3.3.4
		pip install pandas==1.1.5

Setting up the Python virtual environment typically takes around 40 seconds, minimizing any significant impact on solution time.

Following the default Ansys Mechanical Command, append '&' and the commands to repeatedly invoke the convergence script every 5 minutes. Placing '&' after the Ansys run command runs the task in the background. The Linux process ID of the Ansys process is captured, and as long as it is active (indicating the model is running), the convergence.py script is repeatedly called to update the convergence.png file within the work/shared directory.

Uncomment the appropriate LICENSE_FEATURE value based on the intended license to be used.

Finally, the convergence script is called one last time after the Ansys solver completes. This ensures that the convergence graph is updated with the latest iteration data point before the cluster is shut down. Any necessary post-processing commands can be added after the deactivate command.

		export LICENSE_FEATURE=ansys
		#export LICENSE_FEATURE=meba
		#export LICENSE_FEATURE=mech_1
		#export LICENSE_FEATURE=mech_2
		ansys${ANSYSMECH_VERSION/./} -dis -b -mpi $ANS_MPI_VERS -np $RESCALE_CORES_PER_SLOT -machines $MACHINES -i *.dat -p $LICENSE_FEATURE &
		pid=$!
		while ps -p $pid &>/dev/null; do
		sleep 300
		python convergence.py >> convergence.log
		done
		python convergence.py >> convergence.log
		deactivate

In your software command box, the ideal arrangement would resemble the following:

![Screenshot 2023-08-14 at 5 20 42 PM](https://github.com/rescale/soln-wee-scripts/assets/64439634/e525e19b-1943-48f9-b595-78b1f9c6cf70)

Users can easily access the convergence.png file for review during the job's execution by performing a search in the livetailing menu. This provides a convenient way to monitor the Force Convergence Graph in real-time as the simulation progresses.

https://github.com/rescale-labs/Util_Ansys_Mech_Convergence_Plotter/assets/64439634/26b65b78-1c03-4706-97b7-ed80cad5367c

## Using Jupyter Notebook

For Rescale accounts configured to employ Jupyter Notebook alongside Jobs, users have the capability to execute the convergence.py file directly from within the Jupyter Notebook interface, facilitating the visualization of the graph.

To achieve this:

1. Ensure that convergence.py is included among the Job inputs, or utilize a template that incorporates this file within the Inputs section.

2. Initiate the Jupyter Notebook by accessing it through the Status Page.

3. Navigate to the Shared directory and establish a new Python3 Session.

4. Within the notebook cell, input the command %run convergence.py, and subsequently execute it by pressing Run (Shift + Enter). This action will trigger the generation of the convergence.png plot, allowing for immediate visualization.

https://github.com/rescale-labs/Util_Ansys_Mech_Convergence_Plotter/assets/64439634/1ffe487d-f0b9-4b83-9905-a5ab6b2d03a4

# Edits

Users can review the python script to adapt this for other convergence plots such as Displacement Convergence.

To adjust the size of the plot to better fit your monitor screen size you can edit line #35. 
Here 12,7 represents the width x height of the plot in inches.

fig, ax = plt.subplots(2, 1, figsize=(12, 7), sharex=True, gridspec_kw={'height_ratios': [4, 1]}) 

# Support
This script has been tested to work with the following Ansys Mechanical Versions?

2023 R2
2023 R1
2022 R2
2022 R1

