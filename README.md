# CSharp_DotNet-Final-Project
My final project for the C# and .NET course at The Tech Academy was to create a Rental History page with CRUD functionality as part of a software built for a local theater. The Rental History Index page shows information on past rentals organized in a table. By clicking on "Details", users can see more information about a chosen item. Only the administrator can create new items or edit/delete an existing item. I worked on several back end but also front end stories, using Visual Studio, the .NET Framework, MVC, C#, JavaScript, jQuery, HTML, CSS and Bootstrap. I worked on a team with other students responsible for other parts of the same project.

Below are descriptions of the stories I worked on, along with code snippets:

# Back End Stories:

## Creating an entity model for the Rental History page and developing CRUD pages using the Code-First approach
     
     As a start of the Rental History page, I created an entity model for the Rental History class, so it can 
     be saved to the database. First I created a model, then I created CRUD (Index, Create, Edit, Update and 
     Delete) pages for it using Entity Framework.

## Adding a sorting feature on the Index page

    The rentals will first appear from most recent to oldest. Users are able to filter the list by only seeing 
    the damaged/undamaged rentals. They can also sort the list by Rental Name in ascending or descending order 
    (A - Z or Z - A). Selecting one of these options does not reload the page. I was using Ajax to contact a 
    controller method to achieve this.

    Index.cshtml:

         <table class="table RentalHistory-index--table">
         <tr>
             <td colspan="3">
                 Most recent Rental Histories
             </td>
             <td colspan="2" id="RentalHistory-index--SortedBy">
                 <label for="sortingCriteria">Sorted by:</label>
                 <select id="sortingSelect" class="form-control" name="sortingCriteria">
                     <option value="nosorting">No Extra Sorting...</option>
                     <option value="damagedrentals">Damaged Rentals</option>
                     <option value="undamagedrentals">Undamaged Rentals</option>
                     <option value="rentalsAtoZ">Rentals A to Z</option>
                     <option value="rentalsZtoA">Rentals Z to A</option>
                 </select>
             </td>
         </tr>
         @Html.Partial("_RentalHistoriesPartial")
         </table>

    Controller Index method sorts the rentals by newest entry first:
     
         public ActionResult Index()
         {
                 return View(db.RentalHistories.OrderByDescending(item => item.RentalHistoryId).ToList());
     
         }

    Adding new Controller methods:
    
        public ActionResult SortByDamaged()
        {
            var sortedRentals = db.RentalHistories.Where(r => r.RentalDamaged).ToList();
            return PartialView("_RentalHistoriesPartial", sortedRentals);
        }
        public ActionResult SortByUndamaged()
        {
            var sortedRentals = db.RentalHistories.Where(r => !r.RentalDamaged).ToList();
            return PartialView("_RentalHistoriesPartial", sortedRentals);
        }
        public ActionResult SortByAZ()
        {
            var sortedRentals = db.RentalHistories.OrderBy(r => r.Rental).ToList();
            return PartialView("_RentalHistoriesPartial", sortedRentals);
        }
        public ActionResult SortByZA()
        {
            var sortedRentals = db.RentalHistories.OrderByDescending(r => r.Rental).ToList();
            return PartialView("_RentalHistoriesPartial", sortedRentals);
        }

        public ActionResult NoSorting()
        {
            var sortedRentals = db.RentalHistories.OrderByDescending(item => item.RentalHistoryId).ToList();
            return PartialView("_RentalHistoriesPartial", sortedRentals);
        }
    
## Creating a seed method for the History Manager or admin

    I created an instance of the History Manager and saved it in the database so we can access the History Manager 
    for testing purposes. I created a user role using the classes UserManager and RoleManager, named it 
    "HistoryManager" and assigned it to the History Manager being seeded.

    HistoryManager.cs:

    public static void SeedHistoryManager(ApplicationDbContext context)
        {
            var roleManager = new RoleManager<IdentityRole>(new RoleStore<IdentityRole>(context));
            var userManager = new UserManager<ApplicationUser>(new UserStore<ApplicationUser>(context));

            if (!roleManager.RoleExists("HistoryManager"))
            {
                var role = new IdentityRole();
                role.Name = "HistoryManager";
                roleManager.Create(role);
                var historymanager = new HistoryManager
                {
                    UserName = "historymanagerusername1",
                    Email = "historymanagerusername1@historymanager.com",
                    PhoneNumber = "5034445555"
                };

                var result = userManager.Create(historymanager, "passwordtest6677");

                if (result.Succeeded)
                {
                    result = userManager.AddToRole(historymanager.Id, "HistoryManager");
                    string newId = historymanager.Id;
                }
            }
        }

## Restricting CRUD pages to admin only

    The Create, Edit, and Delete functionality is restricted to the History Manager. Users are redirected to an Access 
    Denied page with a link to the login page, when they try to access one of these pages. For testing purposes, 
    I created an easily accessible login button that automatically signs the user in as a History Manager seeded in the 
    database, when clicked. The button is fixed to the bottom right side of the screen and only appears on the Index 
    and Details pages, additionally it is not displayed when the current user is logged in.

    HistoryManager.cs:

    public class HistoryManagerAuthorize : AuthorizeAttribute
    { 
       public override void OnAuthorization(AuthorizationContext filterContext)

       {
           base.OnAuthorization(filterContext);

           if (filterContext.Result is HttpUnauthorizedResult)
           {
               filterContext.Result = new RedirectResult("~/Rent/RentalHistories/AccessDenied");
           }
       }
     }

     Partial View for the login button:
     
     @using (Html.BeginForm("HistoryManagerLogin", "Account", new { area = "", ReturnUrl = Request.Url.AbsoluteUri }, FormMethod.Post))
     {
     @Html.AntiForgeryToken()
     <button class="btn btn-danger" value="Log in" type="submit" id="RentalHistory-index--HistoryManagerLoginBtn">
        Log in as History Manager</button>
     }

     This line of code is added above the Create, Edit and Delete methods in the RentalHistoriesController:

     [HistoryManager.HistoryManagerAuthorize(Roles = "HistoryManager")]
     

# Front End Stories:

## Styling the Create and Edit pages with CSS, Bootstrap and JavaScript
  
    If a Rental is not damaged, so the checkbox is not clicked, the "Damages Incurred" label changes to the 
    text "Notes".

## Styling the Index page with CSS and Bootstrap, plus JavaScript 
The history of rentals is shown in a tabular form. If a Rental is damaged, a red X symbol is shown next to it, if it's 
not, there is a green checkmark symbol. I used Font-Awesome for these icons. The name of the rental items is displayed 
using Bootstrap badge, so it can be distinguished. If the rental item is not damaged the description of the damage/notes 
is greyed out. Here I'm using ellipses so long text is cut off instead of wrapping to a new line. Users can see more 
information about the damage (or any notes) by going to the item's Details page. When hovering over a row, vertical 
ellipses appear for the dropdown menu containing Edit, Details and Delete with related icons.

## Collapse feature to show details on the Index page using Bootstrap 
Users can open the accordion that show details of the damage or the notes on the rental.

