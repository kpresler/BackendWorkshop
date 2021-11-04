# Backend Implementation Workshop

In this workshop, you'll start implementing the backend of the CoffeeMaker system.  We'll walk you through some initial examples to get you started and then you'll implement your backend design, which will include the creation of your persistent classes and your REST API. 

You will also test your backend to ensure that everything works correctly using the strategies discussed in the Testing Workshop.

## Activity: Team Workshop

We have the backend implementation and testing workshop into two parts, that you should complete in order.  These will help your with your Team Technical Deliverables and your Independent Technical Deliverables.

  * [Part 1: Database and Persistence Implementation](persistence-implementation.md)
  * [Part 2: REST API Implementation](api-implementation.md)


## Activity: Team Technical Deliverables

**_Deliver your technical deliverables to the `development` branch.  DO NOT PR to `main` until we do the frontend next week!_**

Together with your team, implement the persistence and API logic for your backend design around the arbitrary ingredients functionality.  Note that the design discussed in the [Part 1: Database and Persistence Implementation](persistence-implementation.md) portion of the website is _not sufficient_ because the enumerated types do not support dynamic ingredient types when adding or editing recipes.  


You will **implement and test** your design for the _arbitrary ingredient_ functionality. You are welcome to draw inspiration from the [Part 1: Database and Persistence Implementation](persistence-implementation.md) workshop, which may lead to a change in your original design.  As you implement, you may decide that you original design was not sufficient.  **You should document these implementation decisions in a wiki page called "Implementation Decisions".**

  * Persistence Done Criteria
     * New arbitrary ingredients can be created, saved to the database, retrieved, updated, and deleted.
	 * The existing Inventory and Recipe classes are modified (which may include removal of existing classes depending on your design) to incorporate the new ingredients functionality.
	 * 70% line/statement coverage across all persistence classes.
  * REST API Done Criteria
     * New API endpoints for interacting with the ingredients functionality (add, update, delete, get all).
	 * Updates to existing endpoints for `Inventory` and `Recipe` to include new functionality and meet requirements (EXCEPT for Edit Recipes functionality (UC4)).
	 * 70% line/statement coverage across all controller classes.
  * Update provided tests so they compile and pass. 	

## Activity: Independent Technical Deliverables

**_Deliver your technical deliverables to the development branch.  DO NOT PR to main!_**

You will integrate your team's implementation and tests into your individual CoffeeMaker project.  We suggest you do this on the file system and then refresh the project in Eclipse to see the changes.

You will **implement and test** your design for the Edit Recipe functionality.

  * Persistence Done Criteria
     * Edit recipe is implemented and tested including arbitrary ingredients.
	 * 70% line/statement coverage across all persistence classes.
  * REST API Done Criteria
     * End points associated with the edit recipe requirements are updated to incorporate the arbitrary ingredients functionality.
	 * 70% line/statement coverage across all controller classes.
  * Backend Design Reflection
     * In your wiki, create a page called "Backend Design Reflection" and reflect on your team and individual backend designs and implementations.  Discuss any changes that were made during implementation.  Additionally, reflect on how the `Recipe` class design at the team level impacted your implementation of the edit recipe functionality.
  * Update provided tests so they compile and pass (you can copy these from the team repo).