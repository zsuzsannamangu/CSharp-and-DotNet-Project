# CSharp_DotNet-Final-Project
My final project for the C# and .NET course at The Tech Academy was to create a Rental History page with CRUD functionality as part of a software built for a local theater. The Rental History Index page shows information on past rentals organized in a table. By clicking on "Details", users can see more information about a chosen item. Only the administrator can create new items or edit/delete an existing item. I worked on several back end but also front end stories, using Visual Studio, the .NET Framework, MVC, C#, JavaScript, jQuery, HTML, CSS and Bootstrap. I worked on a team with other students responsible for other parts of the same project.

Below are descriptions of the stories I worked on, along with code snippets:

# Back End Stories:

## Creating an entity model for the Rental History page and developing CRUD pages using the Code-First approach
     
As a start of the Rental History page, I created an entity model for the Rental History class, so it can 
be saved to the database. First I created a model, then I created CRUD (Index, Create, Edit, Update and 
Delete) pages for it using Entity Framework.

     RentalHistoriesController.cs:

          using System.Data;
          using System.Data.Entity;
          using System.Linq;
          using System.Net;
          using System.Web.Mvc;
          using TheatreCMS3.DAL;
          using TheatreCMS3.Areas.Rent.Models;
          
          namespace TheatreCMS3.Areas.Rent.Controllers
          {
              public class RentalHistoriesController : Controller
              {
                  private RentalHistoryContext db = new RentalHistoryContext(); //a class variable has been automatically created that instantiates a database context object
          
                  // GET: RentalHistories
                  public ActionResult Index() //the index action method gets a rental history list
                  {
                      return View(db.RentalHistories.OrderByDescending(item => item.RentalHistoryId).ToList());
          
                  }
          
          
                  // GET: RentalHistories/Details/5
                  public ActionResult Details(int? id)
                  {
                      if (id == null)
                      {
                          return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
                      }
                      RentalHistory rentalHistory = db.RentalHistories.Find(id);
                      if (rentalHistory == null)
                      {
                          return HttpNotFound();
                      }
                      return View(rentalHistory);
                  }
          
          
                  // GET: RentalHistories/Create
                  [HistoryManager.HistoryManagerAuthorize(Roles = "HistoryManager")]
                  public ActionResult Create()
                  {
                      return View();
                  }
          
                  // POST: RentalHistories/Create
                  // To protect from overposting attacks, enable the specific properties you want to bind to, for 
                  // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
                  [HistoryManager.HistoryManagerAuthorize(Roles = "HistoryManager")]
                  [HttpPost]
                  [ValidateAntiForgeryToken]
                  public ActionResult Create([Bind(Include = "RentalHistoryId,RentalDamaged,DamagesIncurred,Rental,DateCreated")] RentalHistory rentalHistory)
                  {
                      try
                      {
                          if (ModelState.IsValid)
                          {
                              db.RentalHistories.Add(rentalHistory);
                              db.SaveChanges();
                              return RedirectToAction("Index");
                          }
                      }
                      catch (DataException)
                      {
                          ModelState.AddModelError("", "Unable to save changes. Try again, and if the problem persists, contact your system administrator.");
                      }
                      return View(rentalHistory);
                  }
          
                  // GET: RentalHistories/Edit/5
                  [HistoryManager.HistoryManagerAuthorize(Roles = "HistoryManager")]
                  public ActionResult Edit(int? id)
                  {
                      if (id == null)
                      {
                          return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
                      }
                      RentalHistory rentalHistory = db.RentalHistories.Find(id);
                      if (rentalHistory == null)
                      {
                          return HttpNotFound();
                      }
                      return View(rentalHistory);
                  }
          
                  [HistoryManager.HistoryManagerAuthorize(Roles = "HistoryManager")]
                  // POST: RentalHistories/Edit/5
                  // To protect from overposting attacks, enable the specific properties you want to bind to, for 
                  // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
                  [HttpPost, ActionName("Edit")]
                  [ValidateAntiForgeryToken]
                  public ActionResult Edit([Bind(Include = "RentalHistoryId,RentalDamaged,DamagesIncurred,Rental")] RentalHistory rentalHistory)
                  {
                      if (ModelState.IsValid)
                      {
                          try
                          {
                              db.Entry(rentalHistory).State = EntityState.Modified;
                              db.SaveChanges();
                              return RedirectToAction("Index");
                          }
                          catch (DataException)
                          {
                              ModelState.AddModelError("", "Unable to save changes. Try again, and if the problem persists, contact your system administrator.");
                          }
                      }
                      return View(rentalHistory);
                  }
          
                  // GET: RentalHistories/Delete/5
                  [HistoryManager.HistoryManagerAuthorize(Roles = "HistoryManager")]
                  public ActionResult Delete(int? id, bool? saveChangesError=false)
                  {
                      if (id == null)
                      {
                          return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
                      }
                      if (saveChangesError.GetValueOrDefault())
                      {
                          ViewBag.ErrorMessage = "Delete failed. Try again, and if the problem persists, contact your system administrator.";
                      }
                      RentalHistory rentalHistory = db.RentalHistories.Find(id);
                      if (rentalHistory == null)
                      {
                          return HttpNotFound();
                      }
                      return View(rentalHistory);
                  }
          
                  // POST: RentalHistories/Delete/5
                  [HistoryManager.HistoryManagerAuthorize(Roles = "HistoryManager")]
                  [HttpPost, ActionName("Delete")]
                  [ValidateAntiForgeryToken]
                  public ActionResult Delete(int id)
                  {
                      try
                      {
                          RentalHistory rentalHistory = db.RentalHistories.Find(id);
                          db.RentalHistories.Remove(rentalHistory);
                          db.SaveChanges();
                      }
                      catch (DataException)
                      {
                          return RedirectToAction("Delete", new { id = id, saveChangesError = true });
                      }
                      
                      return RedirectToAction("Index");
                  }
          
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
                  protected override void Dispose(bool disposing) //this closes database connection and frees up resources
                  {
                      if (disposing)
                      {
                          db.Dispose();
                      }
                      base.Dispose(disposing);
                  }
          
                  public ActionResult AccessDenied()
                  {
                      return View();
                  }
          
                  public ActionResult _LoginBtnHistoryManager()
                  {
                      return PartialView();
                  }
              }
          }

     RentalHistory.cs (the Model):

          namespace TheatreCMS3.Areas.Rent.Models
          {
              public class RentalHistory
              {
                  public int RentalHistoryId { get; set; }
                  public bool RentalDamaged { get; set; }
                  public string DamagesIncurred { get; set; }
                  public string Rental { get; set; }
              }
          }


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

     _Layout.cshtml:
     
          @*The code below checks what controller the current view is. If it's a RentalHistories view than it will display the Easy Login button for HistoryManager*@
         @if (HttpContext.Current.Request.RequestContext.RouteData.Values["controller"].ToString() == "RentalHistories")
         {
             if (!Request.IsAuthenticated)
             {
                 @Html.Partial("_LoginBtnHistoryManager")
             }
         }

     This code is added in the AccountController.cs file, which controlls the login page:
     
          [HttpPost]
          [AllowAnonymous]
          [ValidateAntiForgeryToken]
          
          public async Task<ActionResult> HistoryManagerLogin(string returnUrl)
          {
            var result = await SignInManager.PasswordSignInAsync("historymanagerusername1", "passwordtest6677", true, shouldLockout: false);
            switch (result)
            {
                case SignInStatus.Success:
                    return RedirectToLocal(returnUrl);
                case SignInStatus.LockedOut:
                    return View("Lockout");
                case SignInStatus.RequiresVerification:
                    return RedirectToAction("SendCode", new { ReturnUrl = returnUrl, RememberMe = true });
                case SignInStatus.Failure:
                default:
                    ModelState.AddModelError("", "Invalid login attempt.");
                    return View();
            }
          }

     The AccessDenied.cshtml view:

          <h2>Access Denied</h2>
          <p>You are not authorized to visit this page!</p>
          <div class="RentalHistory-AccessDenied-image">
               <img src="~/Content/images/rentalhistory_unauthorized.jpg" alt="Access denied">
          </div>
          <div class="RentalHistory-AccessDenied-button">
               <button class="btn btn-secondary m-3 RentalHistory-backToList-button">@Html.ActionLink("Click here to login", "../../Account/Login")</button>
          </div>
     

# Front End Stories:

## Styling the Create and Edit pages with CSS, Bootstrap and JavaScript
  
If a Rental is not damaged, so the checkbox is not clicked, the "Damages Incurred" label changes to the 
text "Notes".

     Create.cshtml:

          @model TheatreCMS3.Areas.Rent.Models.RentalHistory
          
          @{
              ViewBag.Title = "Create";
              Layout = "~/Views/Shared/_Layout.cshtml";
          }
          
          <div class="text-center RentalHistory-create--title">
              <h1>Create Rental History</h1>
          </div>
          <div class="m-3">
              <div class="card col-sm-14 col-md-12 col-lg-8 mx-auto shadow RentalHistory-create--form">
                  <div class="card-body">
                      @using (Html.BeginForm())
                      {
                          @Html.AntiForgeryToken()
          
                          <div>
                              @Html.ValidationSummary(true, "", new { @class = "text-danger" })
          
                              <form class="row g-3">
                                  <div class="form-group row">
                                      <div class="col-sm-6">
                                          @Html.LabelFor(model => model.Rental, htmlAttributes: new { @class = "form-label" })
                                          @Html.EditorFor(model => model.Rental, new { htmlAttributes = new { @class = "form-control" } })
                                          @Html.ValidationMessageFor(model => model.Rental, "", new { @class = "text-danger" })
                                      </div>
                                      <div class="form-check col-sm-6" onclick = "labelNameChangeFunction()">
                                          @Html.EditorFor(model => model.RentalDamaged, new { htmlAttributes = new { @class = "form-check-input checkbox mt-sm-5 ml-sm-4", id = "rentalDamageCheckbox" } })
                                          @Html.ValidationMessageFor(model => model.RentalDamaged, "", new { @class = "text-danger" })
                                          @Html.LabelFor(model => model.RentalDamaged, htmlAttributes: new { @class = "form-check-label"})
                                      </div>
                                  </div>
                                  <div class="form-group row">
                                      <div class="col-12">
                                          <label id="RentalHistory-create--notesLabel">Notes</label>
                                          <div id="RentalHistory-create--damagesIncurredLabel">@Html.LabelFor(model => model.DamagesIncurred, new { htmlAttributes = new { @class = "form-label" } })</div>
                                          @Html.EditorFor(model => model.DamagesIncurred, new { htmlAttributes = new { @class = "form-control" } })
                                          @Html.ValidationMessageFor(model => model.DamagesIncurred, "", new { @class = "text-danger" })
                                      </div>
                                  </div>
          
                                  <div class="form-group row">
                                      <div class="mx-auto">
                                          <input type="submit" value="Create" class="btn btn-success mx-auto" />
                                          <button class="btn btn-secondary RentalHistory-backToList-button">@Html.ActionLink("Back to List", "Index")</button>
                                      </div>
                                  </div>
          
                              </form>
                          </div>
                      }
                  </div>
              </div>
          </div>

     Edit.cshtml:

          @model TheatreCMS3.Areas.Rent.Models.RentalHistory
          @Styles.Render("~/Content/Areas/Rent.css")
          
          @{
              ViewBag.Title = "Edit";
              Layout = "~/Views/Shared/_Layout.cshtml";
          }
          
          
          <div class="text-center RentalHistory-create--title">
              <h1>Edit Rental History</h1>
          </div>
          <div class="m-3">
              <div class="card col-sm-14 col-md-12 col-lg-8 mx-auto shadow RentalHistory-create--form">
                  <div class="card-body">
                      @using (Html.BeginForm())
                      {
                          @Html.AntiForgeryToken()
          
                          <div>
                              @Html.ValidationSummary(true, "", new { @class = "text-danger" })
                              @Html.HiddenFor(model => model.RentalHistoryId)
          
                              <form class="row g-3">
                                  <div class="form-group row">
                                      <div class="col-sm-6">
                                          @Html.LabelFor(model => model.Rental, htmlAttributes: new { @class = "form-label" })
                                          @Html.EditorFor(model => model.Rental, new { htmlAttributes = new { @class = "form-control" } })
                                          @Html.ValidationMessageFor(model => model.Rental, "", new { @class = "text-danger" })
                                      </div>
                                      <div class="form-check col-sm-6" onclick = "labelNameChangeFunction()">
                                          @Html.EditorFor(model => model.RentalDamaged, new { htmlAttributes = new { @class = "form-check-input checkbox mt-sm-5 ml-sm-4", id = "rentalDamageCheckbox" } })
                                          @Html.ValidationMessageFor(model => model.RentalDamaged, "", new { @class = "text-danger" })
                                          @Html.LabelFor(model => model.RentalDamaged, htmlAttributes: new { @class = "form-check-label" })
                                      </div>
                                  </div>
                                  <div class="form-group row">
                                      <div class="col-12">
                                          <label id="RentalHistory-create--notesLabel">Notes</label>
                                          <div id="RentalHistory-create--damagesIncurredLabel">@Html.LabelFor(model => model.DamagesIncurred, htmlAttributes: new { @class = "form-label" })</div>
                                          @Html.EditorFor(model => model.DamagesIncurred, new { htmlAttributes = new { @class = "form-control" } })
                                          @Html.ValidationMessageFor(model => model.DamagesIncurred, "", new { @class = "text-danger" })
                                      </div>
                                  </div>
          
                                  <div class="form-group row">
                                      <div class="mx-auto">
                                          <input type="submit" value="Save" class="btn btn-success mx-auto" />
                                          <button class="btn btn-secondary RentalHistory-backToList-button">@Html.ActionLink("Back to List", "Index")</button>
                                      </div>
                                  </div>
          
                              </form>
                          </div>
                      }
                  </div>
              </div>
          </div>


## Styling the Index page with CSS and Bootstrap, plus JavaScript 
The history of rentals is shown in a tabular form. If a Rental is damaged, a red X symbol is shown next to it, if it's 
not, there is a green checkmark symbol. I used Font-Awesome for these icons. The name of the rental items is displayed 
using Bootstrap badge, so it can be distinguished. If the rental item is not damaged the description of the damage/notes 
is greyed out. Here I'm using ellipses so long text is cut off instead of wrapping to a new line. Users can see more 
information about the damage (or any notes) by going to the item's Details page. When hovering over a row, vertical 
ellipses appear for the dropdown menu containing Edit, Details and Delete with related icons. There is an accordion
feature that shows or hides the details of the damage (or the notes).

     Rental Histories Partial View:

          @model IEnumerable<TheatreCMS3.Areas.Rent.Models.RentalHistory>
          
          <table class="table RentalHistory-index--table borderless" id="RentalHistory-index--tableId">
              @foreach (var item in Model)
              {
              <tr class="RentalHistory-index--tr">
                  <td id="RentalHistory-index--itemId">
                  @Html.DisplayFor(modelItem => item.RentalHistoryId)
                  </td>
                  <td>
                  @if (item.RentalDamaged == true)
                  {
                      <p><i class="fa-solid fa-circle-xmark" style="color:red"></i></p>
                  }
                  else
                  {
                      <p><i class="fa-solid fa-circle-check" style="color:green"></i></p>
                  }
                  </td>
                  <td>
                  <span class="badge badge-dark RentalHistory-index--rental">@Html.DisplayFor(modelItem => item.Rental)</span>
                  </td>
                  <td class="btn RentalHistory-index--damageNotesButton" type="button" data-toggle="collapse" data-target="#collapse_@item.RentalHistoryId">
                  @if (item.RentalDamaged == true)
                  {
                      <span class="text-muted d-inline-block text-truncate" style="max-width: 600px;">Damage details</span>
                  }
                  else
                  {
                      <span class="text-dark d-inline-block text-truncate" style="max-width: 600px;">None</span>
                  }
                  </td>
                  <td>
                  <div id="collapse_@item.RentalHistoryId" class="collapse">
                      @if (item.RentalDamaged == true)
                      {
                      <span class="text-muted d-inline-block text-truncate" style="max-width: 600px;">@Html.DisplayFor(modelItem => item.DamagesIncurred)</span>
                      }
                      else
                      {
                      <span class="text-dark d-inline-block text-truncate" style="max-width: 600px;">@Html.DisplayFor(modelItem => item.DamagesIncurred)</span>
                      }
                  </div>
                  </td>
                  <td>
                  <div class="dropdown">
                      <button class="btn btn-secondary dropdown-toggle RentalHistory-index--dropdownMenuButton" type="button" id="dropdownMenuButton_@item.RentalHistoryId" data-toggle="dropdown" aria-haspopup="true">
                      <i class="fas fa-ellipsis-vertical"></i>
                      </button>
                      <div class="dropdown-menu" aria-labelledby="dropdownMenuButton_@item.RentalHistoryId">
                      <span class="dropdown-item">
                          <i class="fas fa-pen mx-2"></i> @Html.ActionLink("Edit", "Edit", new { id = item.RentalHistoryId })
                      </span>
                      <span class="dropdown-item">
                          <i class="fas fa-info-circle mx-2"></i> @Html.ActionLink("Details", "Details", new { id = item.RentalHistoryId })
                      </span>
                      <span class="dropdown-item" style="color:red">
                          <i class="fas fa-trash mx-2" style="color:red"></i> @Html.ActionLink("Delete", "Delete", new { id = item.RentalHistoryId })
                      </span>
                      </div>
                  </div>
                  </td>
              </tr>
              }
          </table>

## Creating and styling the Donation page
Users can use this form to enter their information and the donation amount. The input fields have placeholders. I'm using Bootstrap's Grid layout for the layout of the page.

     Donate.cshtml view:

          @{
              ViewBag.Title = "Donate";
          }
          
          <div class="text-center donate-index--title">
              <h1>Consider Making a Donation</h1>
          </div>
          
          <div class="row no-gutters m-3">
              <div class="card col-sm-14 col-md-12 col-lg-8 mx-auto shadow donate-index--form">
                  <div class="card-body">
                      <!--Html.BeginForm needs the controller method information that can be called when user submits the form-->
                      @using (Html.BeginForm("", "", FormMethod.Post))
                      {
                          <form class="container">
                              <div class="form-group row">
                                  <label for="Name" class="col-sm-4 col-form-label">Name*</label>
                                  <div class="input-group col-sm-8">
                                      <input name="Name" type="text" class="form-control mr-sm-1" placeholder="First Name" />
                                      <input name="Name" type="text" class="form-control ml-sm-1" placeholder="Last Name" />
                                  </div>
                              </div>
                              <div class="form-group row">
                                  <label for="EmailAddress" class="col-sm-4 col-form-label">Email Address*</label>
                                  <div class="col-sm-8">
                                      <input name="EmailAddress" type="email" class="form-control" placeholder="Email Address" />
                                  </div>
                              </div>
                              <div class="form-group row">
                                  <label for="Address" class="col-sm-4 col-form-label">Address</label>
                                  <div class="col-sm-8">
                                      <input name="Address" type="text" class="form-control" placeholder="Address" />
                                  </div>
                              </div>
                              <div class="form-group row">
                                  <label for="DonationAmount" class="col-sm-4 col-form-label">Donation Amount($)*</label>
                                  <div class="col-sm-8">
                                      <input name="DonationAmount" type="number" class="form-control" placeholder="Donation Amount" />
                                  </div>
                              </div>
                              <div class="form-group row">
                                  <label for="Comments" class="col-sm-4 col-form-label">Donation Comments</label>
                                  <div class="col-sm-8">
                                      <textarea name="Comments" type="text" class="form-control" rows="4" placeholder="Place comments here..."></textarea>
                                  </div>
                              </div>
                              <button type="submit" class="btn btn-light">Submit</button>
                          </form>
                      }
                  </div>
              </div>
          </div>
