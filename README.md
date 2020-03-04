# C# Live Project

### Introduction
Doing two-week sprints using different programming languages has really helped solidify my understanding of how full scale MVC web applications work, as well as using functions, and storing important information in a database. For this sprint I have been writing in C#, which is a very powerful language, and can be used for all sorts of applications from smaller console apps to full scale websites.  Being someone who uses a Mac as my primary computer, I decided to test my knowledge with Windows OS this time around, so I'm able to handle any situation. As you continue reading I will explain some challenges I had and how I overcame them to create the most functional UX, as possible. Because, in my opinion, an app is only as good as the experience the user has, no matter how many features it has.

Below are some examples of features I implemented.

### Stories
[Subscriber Dashboard](#Subscriber-Dashboard)\
[Saving Sponsor Logos](#Saving-Sponsor-Logos)\
[Display logos](#Display-logos)

I concentrated a lot of my efforts on back end and functional aspects of the website. I wanted my product to work as smoothly as possible, creating functional flow of webpages, and displaying information the user wanted. It was important to me to create simple and readable code, not only for efficiency, but for ease of trouble shooting and maintenance.

#### Subscriber Dashboard
One of the first back end stories I worked on was the ability for the user to access the subscriber dashboard even without a subscription yet. Allowing the user to then subscribe and get notifications, or manage other aspects of their account. This was a fun one because I had a chance to make the dashboard dynamic showing the user if they already had a subscription or if they still needed to sign up. The pictures below show the dashboard with a subscription and without a subscription. 


```
<h2>Welcome to the Subscriber Area!</h2>

<div class="d-flex mt-5">
    <div class="align-self-stretch bg-info text-white col-md-4 mr-1" style="height:402px; border-radius:6px">
        <div class="d-inline-flex flex-row">
            <img src="" class="img-fluid" />
            <h3 class="align-items-center">Upcoming Shows</h3>
        </div>
        <div class="card bg-secondary mt-4 mb-3" style="max-width: 27rem;">
            <div class="card-body">
                <ul>
                    <li>1</li>
                    <li class="overflow-hidden"></li>
                    <li>2</li>
                    <li class="overflow-hidden"></li>
                    <li>3</li>
                    <li class="overflow-hidden"></li>
                    <li>4</li>
                    <li class="overflow-hidden"></li>
                    <li>5</li>
                </ul>
            </div>
        </div>
    </div>
    <div class="d-flex flex-column">
        <div class="p-2 align-self-start bg-info text-white mr-1" style="height:200px; width:200px; border-radius:6px;">
            <div class="d-inline-flex">
                @using (Html.BeginForm())
                {
                    @Html.AntiForgeryToken()
                <div class="form-group">
                    @Html.ValidationSummary(true, "", new { @class = "text-danger" })
                    @Html.HiddenFor(model => model.Id)
                    <h3>@User.Identity.GetUserName()</h3>
                    <br />
                        <p>@Model.StreetAddress</p>
                        <p>@Model.PhoneNumber</p>
                        <p>@Model.Email</p>
                        @*<p>@Model.SubscriberPerson.StreetAddress</p>
                        <p>@Model.SubscriberPerson.PhoneNumber</p>
                        <p>@Model.SubscriberPerson.Email</p>*@
                </div>
                }
            </div>
        </div>
        <div class="p-2 align-self-end bg-secondary text-white mt-1 mr-1" style="height:200px; width:200px; border-radius:6px;">
            @if (Model.SubscriberPerson != null)
            {
                <div class="d-inline-flex">
                    <img src="" class="img-fluid" />
                    <h4 class="align-items-center">Manage Subscription</h4>
                    <p></p>
                </div>
                <a href="/Subscribers/Subscriber/Index" class="btn btn-info mt-3">Button</a>
            }
            else
            {
                <div>
                    <h4 class="align-items-center">You dont currently have a subscription</h4>
                </div>
                <a href="/Subscribers/Subscriber/Create" class="btn btn-info mt-3">Add Subscription</a>
            }
        </div>
    </div>
    <div class="d-flex flex-column">
        <div class="p-2 align-self-start bg-secondary text-white mr-1" style="height:200px; width:200px; border-radius:6px;">
            <div class="d-inline-flex">
                <img src="" class="img-fluid" />
                <h4 class="align-items-center">Manage Account</h4>
                <p></p>
            </div>
            <a href="~/Manage/Index" class="btn btn-info mt-3">Button</a>
        </div>
        <div class="p-2 align-self-end bg-info text-white mt-1 mr-1" style="height:200px; width:200px; border-radius:6px;">
            @if (Model.SeasonManagerPerson.Count() != 0)
            {
                <div class="d-inline-flex">
                    <img src="" class="img-fluid" />
                    <h4 class="align-items-center">Manage Season</h4>
                    <p></p>
                </div>
                <a href="/Subscribers/SeasonManager/Index" class="btn btn-secondary mt-3">Button</a>
            }
            else
            {
                <div>
                    <h4 class="align-items-center">No seasons are currently associated with your account</h4>
                </div>
                <a href="/Subscribers/SeasonManager/Create" class="btn btn-secondary mt-3">Add Season</a>
            }
        </div>
    </div>
</div>
```
<hr>

#### Saving Sponsor Logos
Next I worked on utilizing the ability to have sponsors upload a logo of their choosing. For this I had to use a helper method to convert the photo into a byte array to be saved in the database. Then convert the image back to be displayed in the view.

###### Create method for new sponsors
```
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Create([Bind(Include = "SponsorId,Name,Logo,Height,Width")] Sponsor sponsor, HttpPostedFileBase upload)
{
    if (ModelState.IsValid)
    {
        if (upload != null && upload.ContentLength > 0)
        {
            var logo = ImageUploader.ImageBytes(upload, out string _64);
            sponsor.Logo = logo;
        }
        db.Sponsors.Add(sponsor);
        db.SaveChanges();
        return RedirectToAction("Index");
    }

    return View(sponsor);
}
```

The trick for the edit method was to keep the logo image the same if it wasn't what was being edited.
###### Edit method for sponsors
```
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Edit([Bind(Include = "SponsorId,Name,Logo,Height,Width")] Sponsor sponsor, HttpPostedFileBase upload)
{
    if (ModelState.IsValid)
    {
        if (upload != null && upload.ContentLength > 0)
        {
            var logo = ImageUploader.ImageBytes(upload, out string _64);
            sponsor.Logo = logo;
        }
        db.Entry(sponsor).State = EntityState.Modified;
        db.SaveChanges();
        return RedirectToAction("Index");
    }
    return View(sponsor);
}
```
<hr>

#### Display logos
To display the sponsors desired logo I had to convert the image from a byte to a the base64 version to be displayed to user.

```
byte[] thumbBytes = ImageUploader.ImageThumbnail(item.Logo, 28, 28);
var base64 = System.Convert.ToBase64String(thumbBytes);
img = String.Format("data:image/png;base64,{0}", base64);
```
For the sponsorship index page I needed to display the logo only if one existed in the database for that particular sponsor. I did this using HTML helpers and Razor syntax

```
@foreach (var item in Model) {
    <tr>
        <td>
            @Html.DisplayFor(modelItem => item.Name)
        </td>
        <td>
            @{
                string img = "";
                if (item.Logo != null)
                {
                    byte[] thumbBytes = ImageUploader.ImageThumbnail(item.Logo, 28, 28);
                    var base64 = System.Convert.ToBase64String(thumbBytes);
                    img = String.Format("data:image/png;base64,{0}", base64);
                }
            }
            <img src="@img" />
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.Height)
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.Width)
        </td>
        <td>
            @Html.ActionLink("Edit", "Edit", new { id=item.SponsorId }) |
            @Html.ActionLink("Details", "Details", new { id=item.SponsorId }) |
            @Html.ActionLink("Delete", "Delete", new { id=item.SponsorId })
        </td>
    </tr>
}
```
<hr>


### Team Work
For this project teamwork was key, not only using clean version control, but also to help others better understand the project at hand. As I began to understand the project and what was being asked of me better, I took that as an opportunity to help my team gain a better understanding of the project also. Helping others find better ways to trouble shoot their own problems. Especially when starting a project with new people, I feel like clear communication is key. We succeed as a team or fail as a team.
