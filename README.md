Code summary C# 

During this sprint I attended the sprint planning meeting as well as daily stand ups monday through friday. We worked on a web site for a construction company to help with scheduling and project management. I came in late in the project and worked on it for a little over a week. For this project we use MVC 5 and Bootstrap 3. 

The first user story I worked was to take the index for jobs madel where all the jobs in the jobs table were displayed as a list and display them as cards. As part of this story I was also asked to add the ability to search through the jobs based on any keyword. It would then find where that pattern was matched in any column of the table and return those records. 

The Jobs index.
```c#
@using ConstructionNew.Enums
@using ConstructionNew.Extensions
@model IEnumerable<ConstructionNew.Models.Job>

@{
    ViewBag.Title = "Index";
    var t = TempData["shortMessage"];
}

<h2>Jobs</h2>

<p>
    @Html.ActionLink("Create New", "Create")
</p>

@using (Html.BeginForm())
{
    <p>
        Find: @Html.TextBox("SearchString")
        <input type="submit" value="Search" />
    </p>
}


<div class="row">
    @foreach (var item in Model)
    {

        if (item.JobNumber != null)
        {
            <div class="col-sm-6 col-md-4">
            <div class="thumbnail">
            
                    <div class="caption sub-header">
                   
                        <p class="label green-label pull-left">@Html.DisplayFor(modelItem => item.JobNumber)</p>
                        <span class="h2">
                            @Html.DisplayFor(modelItem => item.JobTitle)
                        </span>
                        <br />
                        <p class="h4">
                            @Html.DisplayFor(modelItem => item.StreetAddress),
                            @Html.DisplayFor(modelItem => item.City),
                            @Html.DisplayFor(modelItem => item.State),
                            @Html.DisplayFor(modelItem => item.Zipcode)
                       
                        </p>
                        <p class="text-right">Note: @Html.DisplayFor(modelItem => item.Note)</p>
                    </div>
            
                    <div>
                        <p class="h3"> Default: @Html.DisplayFor(modelItem => item.ShiftTimes.Default)</p>
                        <p class="h4">
                            Monday: @Html.DisplayFor(modelItem => item.ShiftTimes.Monday),
                            Tuesday: @Html.DisplayFor(modelItem => item.ShiftTimes.Tuesday),
                            Wednesday: @Html.DisplayFor(modelItem => item.ShiftTimes.Wednesday),
                            Thursday: @Html.DisplayFor(modelItem => item.ShiftTimes.Thursday),
                            Friday: @Html.DisplayFor(modelItem => item.ShiftTimes.Friday),
                            Saturday: @Html.DisplayFor(modelItem => item.ShiftTimes.Saturday),
                            Sunday: @Html.DisplayFor(modelItem => item.ShiftTimes.Sunday)
                        </p>
                        <p class="text-right">
                            @if (User.IsInRole("Admin"))
                            {
                                <a class="icon-spacing" href="@Url.Action(" Edit", "Jobs" , new { id=item.JobId }, null)">
                                <span class="glyphicon glyphicon-pencil" aria-hidden="true"></span>
                                </a>
                                <a class="icon-spacing" href="@Url.Action(" Details", "Jobs" , new { id=item.JobId }, null)">
                                <span class="glyphicon glyphicon-info-sign" aria-hidden="true"></span>
                                </a>
                                <a class="icon-spacing" href="@Url.Action(" Delete", "Jobs" , new { id=item.JobId }, null)">
                                <span class="glyphicon glyphicon-trash" aria-hidden="true"></span>
                                </a>
                            }
                            else
                            {
                                <a href="@Url.Action(" Details", "Jobs" , new { id=item.JobId }, null)">
                                <span class="glyphicon glyphicon-info-sign" aria-hidden="true"></span>
                                </a>
                            }
                            </p>
                    </div>
           
            </div>
            </div>
        }
    }
</div>
```

The edited Index controller method. 
```c#
 // GET: Jobs
        [Authorize(Roles = "Admin, Manager")] //used to restrict view to only Admins and Managers
        public ViewResult Index(string searchString)//allows for search function on jobs list
        {
            var jobs = from j in db.Jobs select j;

            if (!String.IsNullOrEmpty(searchString))//if there is no search criteria is given leaves jobs list intact
            {
                jobs = jobs.Where(j => j.JobTitle.Contains(searchString) ||
                                     j.JobNumber.Contains(searchString) ||
                                     j.StreetAddress.Contains(searchString) ||
                                     j.City.Contains(searchString) ||
                                     j.Zipcode.ToString().Contains(searchString) ||
                                     j.State.ToString().Contains( searchString)); //allows to search by state but is restricted to the enum value ex"OR"

            }
            jobs = jobs.OrderBy(i => i.JobNumber);
            return View(jobs.ToList());
        }
```
Next story was to refactor an existing controller method for the schedule model. This method took a list of type schedule and returned a dictionary with a key of jobs and value of schedule from the list given as an argument. The existing method did this with nested for each loops. The concern was the time needed to do as the database scaled. 

Here is the original method.
```c#
 // goes through each schedule in schedule list and, per job, runs through another foreach loop to set a list of schedules and then add that list and job combo to a dictionary.
        protected Dictionary<Job, List<Schedule>> ViewSchedule(List<Schedule> schedules)
        {
            Job job = new Job();
            List<Job> jobs = new List<Job>();
            List<Schedule> dictSchedule = new List<Schedule>();
            var resultDictionary = new Dictionary<Job, List<Schedule>>();
            foreach (Schedule schedule in schedules)
            {
                if (job != schedule.Job && !jobs.Contains(schedule.Job))
                {
                    job = schedule.Job;
                    jobs.Add(job);
                    foreach (Schedule x in schedules)
                    {
                        if (x.Job == job)
                        {
                            dictSchedule.Add(x);
                        }
                    }
                    resultDictionary.Add(job, dictSchedule);
                }
            }
            return resultDictionary;

        }
 ```
Here is my new method. 
```c#
 protected Dictionary<Job, List<Schedule>> ViewSchedule(List<Schedule> schedules)
        {
            var returnDict = new Dictionary<Job, List<Schedule>>();
            
           
            foreach (Schedule schedule in schedules)
            {
                if (returnDict.ContainsKey(schedule.Job))
                {

                    returnDict[schedule.Job].Add(schedule);
                }
                else
                {
                    returnDict[schedule.Job] = new List<Schedule> { schedule };
                }

            }
            return returnDict;

        }
```
The next story I worked on was to rework a search feature on the schedule index at the time is was broken and was set up to allow you to search ether by name or by date. The user request was to allow you to filter schedules by either name or date or both. The code show from the index page is for the most part not my own aside from a few tweaks, that I can’t remember at the time writing this summary, but I felt it helped the context of what is going on in the story. The main change I added was to overloaded the filterReset script to allow the user to search by both name and date. The controller on the other hand is where the issues with this not working were and aside from the method declaration I rewrote the method.

The schedule index.
```c#
@using (Html.BeginForm(FormMethod.Post))
{
    @*Added DropdownList to the "View All" on Employee Schedules*@
    <p>
        @* Filters for searching by user, week or all schedules. Default option is select. Only one filter can have a value at any time - the others will reset *@
        Search by employee: @Html.DropDownList("Person", ((ConstructionNew.Controllers.SchedulesController)this.ViewContext.Controller).GetUsers(), "-- select --",  new { @onclick="filterReset()" } )
        Or filter by week: @Html.DropDownList("SearchWeek", ((ConstructionNew.Controllers.SchedulesController)this.ViewContext.Controller).ListWeeks(), "-- select --", new { @onclick="filterReset()" })
        <button type="submit">Search</button>
        <button type="button" onclick="resetForm()">View All Schedules</button>
    </p>
}
/**********************************************************************************************************************/
                                               There was more code here.
/**********************************************************************************************************************/
<script>
    // If search is filtered by week, reset person selection filter. If search is filtered by person, reset searchweek selection.
    function filterReset() {}
    function filterReset(filterType) {
        if (filterType == 'person') {
            document.getElementById("SearchWeek").selectedIndex = 0;
        } else if (filterType == 'searchWeek') {
            document.getElementById("Person").selectedIndex = 0;
        }
    }
    // Clears both person and week filter to return all schedules
    function resetForm() {
        filterReset('person');
        filterReset('searchWeek');
        document.querySelector("form[action='/Schedules']").submit();
    }
</script>
```
Schedule controller. 
```c#
 [HttpPost]
        public ViewResult Index(string SearchWeek, string Person)
        {   
            var schedList = from s in db.Schedules select s;

            // Filter jobs results by week and by the selected user/employee
            if (Person != "" && SearchWeek != "")
            {
                string[] weekParts = SearchWeek.Split(' ');
                DateTime weekStart = Convert.ToDateTime(weekParts[0]);
                DateTime weekEnd = Convert.ToDateTime(weekParts[2]).AddDays(1);

                schedList = schedList.Where(s => s.Person.Id == Person && (s.StartDate >= weekStart) && (s.StartDate <= weekEnd)
                                 || (s.EndDate <= weekEnd) && (s.EndDate >= weekStart)
                                 || (s.StartDate <= weekStart) && (s.EndDate >= weekEnd)
                                 || (s.StartDate <= weekStart) && (s.EndDate == null));
                return View(ViewSchedule(schedList.ToList()));
            }
            // Filter jobs results by week
            else if (SearchWeek != "")
            {
                string[] weekParts = SearchWeek.Split(' ');
                DateTime weekStart = Convert.ToDateTime(weekParts[0]);
                DateTime weekEnd = Convert.ToDateTime(weekParts[2]).AddDays(1);

                schedList = schedList.Where(s => (s.StartDate >= weekStart) && (s.StartDate <= weekEnd)
                                 || (s.EndDate <= weekEnd) && (s.EndDate >= weekStart)
                                 || (s.StartDate <= weekStart) && (s.EndDate >= weekEnd)
                                 || (s.StartDate <= weekStart) && (s.EndDate == null));

                return View(ViewSchedule(schedList.ToList()));          
            }
            // Filter jobs assigned to the selected user/employee
            else if (Person != "")
            {
                schedList = schedList.Where(s => s.Person.Id == Person);
                
                return View(ViewSchedule(schedList.ToList()));
            }
            // Show all scheduled jobs
            else
            {
                return View(ViewSchedule(db.Schedules.ToList()));
            }                                 
        }
 ```


After the construction project was deemed to be about done we were transitions to a new project. This project was called Management Portal and was to be a remaking of the project we had just finished without a client in mind and a focus on a broader audience without some of the specific design choices made for or by the previous client. 

This allowed me to get the perspective of not just what it's like to come into a project and make the smaller changes that I was making on the construction project but to also see the workload of a new undertaking. 

Since I was finishing up my last story of the construction project my first few stories on the management portal were small fixes from when the views and controllers were first created. Date formatting and Things of that nature. 

The first decent sized story I took was to take the news stories from the CompanyNews table from the database and display them as cards on the home index of the site. 

The home index page
```c#
@model IEnumerable<ManagementPortal.Models.CompanyNews>
@{
   ViewBag.Title = "Home Page";
}


---< there was other code here between the model I included and my code>---- 
<div class="card" >
   <div class="card-header" id="news-header">
       <h3>Company News</h3>
   </div>
   <div class="card-body" id="new-body">
       <div class="row">
           @foreach (var item in Model)
           {
               <div class="col">
                   <div class="card">

                       <div class="card-header">
                           <span class="h4">@Html.DisplayFor(modelItem => item.Title)</span>
                           <span class="pull-right">@Html.DisplayFor(modelItem => item.DateStamp)</span>
                       </div>
                       <div class="card-body">
                           <p>@Html.DisplayFor(modelItem => item.NewsItem)</p>
                       </div>

                   </div>
               </div>
            }
       </div>
   </div>
</div>
```


The changes made to the home controller index method. 
```c#
using ManagementPortal.Models; ←- added this using statement 
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace ManagementPortal.Controllers
{
   public class HomeController : Controller
   {

       private ApplicationDbContext db = new ApplicationDbContext(); ← added this database call as well as modified the method bellow 

       public ActionResult Index()
       {
          
           return View(db.CompanyNews.ToList());
       }
Page continued with code that was not mine.


The styling I added for the cards in shared site.css file. 

/****Company News Cards****/
#news-header {
   background-color: rgb(117, 137, 138);
}

#new-body {
   background-color: rgb(219, 228, 227);
}

#new-body .card {
   box-shadow: 5px 7px var(--dark-grey);
}
```

The next story had me install jQuery UI through the NuGit package manager and use it to create a calendar pop over that would allow you to pick a data and file in text inputs on all applicable form input across the site. 

I added this to several pages across the site about six in total but since there were all generated from the visual studio scaffolding and are all about the same I'll only show one here. 
This is a create form page from CompanyNews app. 
```c#
@model ManagementPortal.Models.CompanyNews

@{
   ViewBag.Title = "Create";
   Layout = "~/Views/Shared/_Layout.cshtml";
}

<h2>Create</h2>

@using (Html.BeginForm())
{
   @Html.AntiForgeryToken()

   <div class="form-horizontal">
       <h4>CompanyNews</h4>
       <hr />
       @Html.ValidationSummary(true, "", new { @class = "text-danger" })
       <div class="form-group">
           @Html.LabelFor(model => model.DateStamp, htmlAttributes: new { @class = "control-label col-md-2" })
           <div class="col-md-10">
               @Html.EditorFor(model => model.DateStamp, new { htmlAttributes = new { @class = "form-control datepicker" } })
               @Html.ValidationMessageFor(model => model.DateStamp, "", new { @class = "text-danger" })
           </div>
       </div>

       <div class="form-group">
           @Html.LabelFor(model => model.Title, htmlAttributes: new { @class = "control-label col-md-2" })
           <div class="col-md-10">
               @Html.EditorFor(model => model.Title, new { htmlAttributes = new { @class = "form-control" } })
               @Html.ValidationMessageFor(model => model.Title, "", new { @class = "text-danger" })
           </div>
       </div>

       <div class="form-group">
           @Html.LabelFor(model => model.NewsItem, htmlAttributes: new { @class = "control-label col-md-2" })
           <div class="col-md-10">
               @Html.EditorFor(model => model.NewsItem, new { htmlAttributes = new { @class = "form-control" } })
               @Html.ValidationMessageFor(model => model.NewsItem, "", new { @class = "text-danger" })
           </div>
       </div>

       <div class="form-group">
           @Html.LabelFor(model => model.ExpirationDate, htmlAttributes: new { @class = "control-label col-md-2" })
           <div class="col-md-10">
          Edit-->     @Html.EditorFor(model => model.ExpirationDate, new { htmlAttributes = new { @class = "form-control datepicker" } })
               @Html.ValidationMessageFor(model => model.ExpirationDate, "", new { @class = "text-danger" })
           </div>
       </div>

       <div class="form-group">
           <div class="col-md-offset-2 col-md-10">
               <input type="submit" value="Create" class="btn btn-default" />
           </div>
       </div>
   </div>
}

<div>
   @Html.ActionLink("Back to List", "Index")
</div>
---EDIT--- 
@section Scripts {
   @Scripts.Render("~/bundles/jqueryui")
}


Here is what I added to the site.js file 

//Datepicker Popup
$(document).ready(function () {
   $('.datepicker').datepicker({

   });

});
```

And the styling added the the site.css file 
```css
/****Datepicker Styling****/

.ui-datepicker {
   background-color: white;
 
   border-radius: 10px;
   width: 200px;
   height: auto;
   margin: 5px auto 0;
  
}

.ui-datepicker table {
   width: 100%;
}

.ui-datepicker-header{
   background-color: var(--light-grey);
}

.ui-datepicker-title {
   text-align: center;
   font-weight: bold;
}

.ui-datepicker-prev {
   float: left;
   background-position: center -30px;
}

.ui-datepicker-next {
   float: right;
   background-position: center 0px;
```

