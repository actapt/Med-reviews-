# Med-reviews-

<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Hospital Feedback Form</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: #f8f9fa;
      padding: 30px;
      max-width: 800px;
      margin: auto;
    }
    h2 {
      color: #007bff;
      text-align: center;
    }
    form {
      background: white;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    }
    label {
      display: block;
      margin: 10px 0 5px;
      font-weight: bold;
    }
    input[type="text"],
    input[type="email"],
    textarea {
      width: 100%;
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 6px;
      font-size: 14px;
    }
    .dept-group {
      background: #f1f1f1;
      border: 1px solid #ccc;
      padding: 12px;
      border-radius: 8px;
      margin-top: 10px;
    }
    .rating-options label {
      display: inline-block;
      margin-right: 12px;
      font-weight: normal;
    }
    button {
      background: #007bff;
      color: white;
      padding: 12px 20px;
      border: none;
      border-radius: 6px;
      font-size: 16px;
      cursor: pointer;
      margin-top: 20px;
    }
    button:hover {
      background: #0056b3;
    }
    #thank-you {
      display: none;
      margin-top: 30px;
      padding: 25px;
      background: #d4edda;
      border: 1px solid #28a745;
      color: #155724;
      border-radius: 8px;
      text-align: center;
    }
    #thank-you a {
      color: #007bff;
      text-decoration: none;
      font-weight: bold;
    }
  </style>
</head>
<body>

  <h2>Hospital Feedback Form</h2>

  <form id="feedback-form">
    <label>Name:
      <input type="text" name="name" required>
    </label>

    <label>Email:
      <input type="email" name="email" required>
    </label>

    <div id="departments-section">
      <p><strong>Select Departments and Rate Each:</strong></p>
    </div>

    <div class="dept-group">
      <strong>Overall Experience Rating:</strong><br>
      <label><input type="radio" name="overall_rating" value="25%" required> 25%</label>
      <label><input type="radio" name="overall_rating" value="50%"> 50%</label>
      <label><input type="radio" name="overall_rating" value="75%"> 75%</label>
      <label><input type="radio" name="overall_rating" value="100%"> 100%</label>
    </div>

    <label>Comments:<br>
      <textarea name="comments" rows="4" placeholder="Enter any suggestions or comments here..."></textarea>
    </label>

    <button type="submit">Submit Feedback</button>
  </form>

  <div id="thank-you">
    <h3>Thank you!</h3>
    <p>Your request has been submitted. We'll get back to you shortly.</p>
    <a href="">Go back to the homepage</a>
  </div>

  <script>
    const departments = [
      "Front Desk", "Administrative Staff", "HMO Staff", "Nurses", "Nurse Assistants",
      "Pharmacy Staff", "Laboratory Staff", "Radiology Staff", "Cleaners", "Kitchen Staff",
      "Security"
    ];

    const departmentsSection = document.getElementById("departments-section");

    departments.forEach((dept, index) => {
      const group = document.createElement("div");
      group.className = "dept-group";
      group.innerHTML = `
        <label><input type="checkbox" name="dept_${index}" value="${dept}"> ${dept}</label>
        <div class="rating-options" id="rating_${index}" style="display:none; margin-top: 5px;">
          <label><input type="radio" name="rating_${index}" value="25%"> 25%</label>
          <label><input type="radio" name="rating_${index}" value="50%"> 50%</label>
          <label><input type="radio" name="rating_${index}" value="75%"> 75%</label>
          <label><input type="radio" name="rating_${index}" value="100%"> 100%</label>
        </div>
      `;
      departmentsSection.appendChild(group);

      group.querySelector(`input[name="dept_${index}"]`).addEventListener("change", function () {
        document.getElementById(`rating_${index}`).style.display = this.checked ? "block" : "none";
      });
    });

    document.getElementById("feedback-form").addEventListener("submit", function (e) {
      e.preventDefault();

      const formData = new FormData(this);
      const now = new Date();
      const date = now.toLocaleDateString();
      const time = now.toLocaleTimeString();

      const data = {
        name: formData.get("name"),
        email: formData.get("email"),
        comments: formData.get("comments"),
        overall_rating: formData.get("overall_rating"),
        date: date,
        time: time
      };

      let deptRatingsText = "";
      departments.forEach((dept, index) => {
        const checkbox = formData.get(`dept_${index}`);
        const rating = formData.get(`rating_${index}`);
        data[dept] = checkbox && rating ? rating : "";
        if (checkbox && rating) {
          deptRatingsText += `${dept}: ${rating}\n`;
        }
      });

      // First submit to SheetDB
      fetch("https://sheetdb.io/api/v1/csg66y63mjr94", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ data })
      })
      .then(() => {
        // Then submit to Formspree with department ratings included
        return fetch("https://formspree.io/f/xgvkqbpp", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            name: data.name,
            email: data.email,
            message:
              `New Hospital Feedback Received:\n\n` +
              `Name: ${data.name}\n` +
              `Email: ${data.email}\n` +
              `Overall Rating: ${data.overall_rating}\n` +
              `Comments: ${data.comments}\n\n` +
              `Department Ratings:\n${deptRatingsText}` +
              `\nSubmitted to: customerservice@sagebits.name.ng`
          })
        });
      })
      .then(() => {
        this.reset();
        departments.forEach((_, index) => {
          document.getElementById(`rating_${index}`).style.display = "none";
        });
        document.getElementById("thank-you").style.display = "block";
        window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' });
      })
      .catch(error => {
        alert("Error submitting form: " + error.message);
      });
    });
  </script>
</body>
</html>
