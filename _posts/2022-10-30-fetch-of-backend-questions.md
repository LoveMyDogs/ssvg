---
title: Fetch of Backend Questions
layout: default
description: Our Frontend talking to Backend Python application serving questions.  This api allows us to get customer responses. 
permalink: /data/quiz
image: /images/feedback.jpeg
tags: [javascript, fetch, dom, getElementID, appendChild]
---
 <div>
   <input type="radio"/>
   
 </div>
<!-- HTML  fragment for page -->
 <div id="result">
    <!-- javascript generated data -->
</div>

<!-- Script is layed out in a sequence (without a function) and will execute when page is loaded -->
<script>

  // prepare HTML defined "result" container for new output
  const resultContainer = document.getElementById("result");

  // prepare fetch urls
  const url = "http://localhost:5000/api/quiz";
  const fetchQuizUrl = '/APStats/5'
  // prepare fetch GET options
  const options = {
    method: 'GET', // *GET, POST, PUT, DELETE, etc.
    mode: 'cors', // no-cors, *cors, same-origin
    cache: 'default', // *default, no-cache, reload, force-cache, only-if-cached
    credentials: 'omit', // include, *same-origin, omit
    headers: {
      'Content-Type': 'application/json'
      // 'Content-Type': 'application/x-www-form-urlencoded',
    },
  };
  // prepare fetch PUT options, clones with JS Spread Operator (...)
  const put_options = {...options, method: 'PUT'}; // clones and replaces method

  currentPageIndex = 0;
  questionIdList = []
  // fetch the API
  fetch(url + fetchQuizUrl, options)
    // response is a RESTful "promise" on any successful fetch
    .then(response => {
      // check for response errors
      if (response.status !== 200) {
          error('GET API response failure: ' + response.status);
          return;
      }
      // valid response will have JSON data
      response.json().then(data => {
          console.log(data);
          onQuizResult(data);
      })
  })
  // catch fetch errors (ie Nginx ACCESS to server blocked)
  .catch(err => {
    error(err + " " + url);
  });

  // Reaction function to likes or jeers user actions
  function onCheckAnswer(type, put_url, elemID) {

    // fetch the API
    fetch(put_url, put_options)
    // response is a RESTful "promise" on any successful fetch
    .then(response => {
      // check for response errors
      if (response.status !== 200) {
          error("PUT API response failure: " + response.status)
          return;  // api failure
      }
      // valid response will have JSON data
      response.json().then(data => {
          console.log(data);
          // TODO: add checkanswer api call here
      })
    })
    // catch fetch errors (ie Nginx ACCESS to server blocked)
    .catch(err => {
      error(err + " " + put_url);
    });
    
  }

  // Create a page for each question. Set first page to shown and subsequent page to 
  // hidden so will be shown when continue button is pressed by the user
  function onQuizResult(questions) {
    index = 0;
    for (const question of questions) {
    
      // make "tr element" for each "row of data"
      const questionDiv = document.createElement("div");
      questionDiv.id = question.id;
      questionIdList.add(questionDiv.id);
      if (index > 0) {
        // Set the first page to shown and rest is not till continue
        // button is pressed
        questionDiv.setAttribute("hidden", true);
      }
      index++;
      if (question.isImage) {
        const desc = document.createElement('use href');
      }
      else {
        questionDiv.innerHTML = question.question;
      } 

      print (question.answer)

      // TODO: loop through choices to make MCs
      const questionButtonDiv = document.createElement("div");
      questionButtonDiv.id= question.id + "choices";
      for (const questionChoice of item.choices) {
        var radioButton = document.createElement("INPUT");
        radioButton.setAttribute("type", "radio");
        var labelValue = document.createElement('label');
        labelValue.innerHTML = questionChoice;
        resultContainer.appendChild(radioButton);
        resultContainer.appendChild(labelValue);
        }
      }

      const questionCheckDiv = document.createElement("div");
      questionCheckDiv.id= question.id + "answer";
      const checkButton = document.createElement('button');
        checkButton.id = question.id + "checkAnswer";
        checkButton.innerHTML = "Check Answer";
        checkButton.onclick = function () {
          // TODO: Call checkanswer rest API ; if score = 0 (result from the API), display incorrect; else display correc
          // how to get question and answer from user to this function
        };
       resultContainer.appendChild(checkButton);  // add "yes button" to yes cell

      const continueButton = document.createElement('button');
        continueButton.id = question.id + "continueButton";
        continueButton.innerHTML = "Continue";
        continueButton.onclick = function () { 
          //TODO: Figure out how to move to next page
          // 1) find next question id and set hidden = false
          // 2) set hidden=true for current page
          // Basically: Hides previous page when you go to another page
          // event.target.parentElement.setAttribute("hidden", true)
          const currentPageId = questionIdList[currentPageIndex];
          currentPageIndex++;
          const nextPageId = questionIdList[currentPageIndex];
          // TODO: currentPageIndex;

        };
        // TODO: another div for solution text
        
        resultContainer.appendChild(checkButton); 

      // Add main div into the dom tree
      resultContainer.appendChild(questionDiv);

      // TODO: remove everything from here since it's for old code
      // td for question cell
      const question = document.createElement("td");
      question.innerHTML = row.id + ". " + row.question;  // add fetched data to innerHTML
  }
  // Something went wrong with actions or responses
  function error(err) {
    // log as Error in console
    console.error(err);
    // append error to resultContainer
    const tr = document.createElement("tr");
    const td = document.createElement("td");
    td.innerHTML = err;
    tr.appendChild(td);
    resultContainer.appendChild(tr);
  }

</script>
