---
title: Quiz Questions
layout: default
description: Our Frontend talking to Backend Python application serving questions.  This api allows us to get customer responses. 
permalink: /data/quizChoices
image: /images/feedback.jpeg
tags: [javascript, fetch, dom, getElementID, appendChild]
---
 
<!-- HTML  fragment for page -->
 <div id="quiz_result">
    <!-- javascript generated data -->
</div>
 
<!-- Script is layed out in a sequence (without a function) and will execute when page is loaded -->
<script>
  const queryString = window.location.search;
  console.log(queryString);
  const urlParams = new URLSearchParams(queryString);
  const subj = urlParams.get('subject');
  const totalQs = urlParams.get('totalQs');

  // prepare HTML defined "result" container for new output
  const resultContainer = document.getElementById("quiz_result");

  // prepare fetch urls
  const url = "https://www.teamcheeseatimetime.tk/api/quiz";
  
  const fetchQuizUrl = `/${subj}/${totalQs}`;
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
  function onCheckAnswer(questId, answer) {

    // const event.target.parentElement.id;
    var requestData = {
      question: questId,
      answer: answer
    };
    const post_options = { 
      ...options, 
      method: 'POST',
      body: JSON.stringify(requestData) 
    }; // clones and replaces method

    post_url = 'checkanswer';
    // fetch the API
    fetch(post_url, post_options)
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
     
      questionIdList.push(questionDiv.id);
      if (index > 0) {
        // Set the first page to shown and rest is not till continue
        // button is pressed
        questionDiv.setAttribute('hidden', true);
      }
      index++;
      var qtitle = document.createElement('div');
      if (question.isImage) {
        var img = document.createElement('img');
        img.src = question.image;
        qtitle.appendChild(img);
      }
      else {
        qtitle.setAttribute(
          'style',
          'color: blue;',
        );
        qtitle.innerHTML = question.question;
      } 
      var hl = document.createElement("hr");
      hl.setAttribute("style", "color:red");
      questionDiv.setAttribute("style", "margin-bottom:20px;");
      questionDiv.appendChild(qtitle);
      questionDiv.appendChild(hl);
     
      const choices = create_choices(question);
      const buttons = create_buttons(question);
      questionDiv.appendChild(choices);
      questionDiv.appendChild(buttons);
      
      resultContainer.appendChild(questionDiv);
      
    } // end of questions
  }
  function create_choices(question)  {
      
      // loop through choices to make MCs
      const choicesDiv = document.createElement("div");
      choicesDiv.id= question.id + "choices";
      answer = '';
      for (const questionChoice of question.choices) {
        var radioDiv = document.createElement("div");
        var radioButton = document.createElement("INPUT");
        radioButton.setAttribute("type", "radio");
        radio.setAttribute(name, choicesDiv.id);
        var labelValue = document.createElement('label');
        labelValue.innerHTML = questionChoice;
        radioDiv.appendChild(radioButton);  
        radioDiv.appendChild(labelValue);
        choicesDiv.appendChild(radioDiv);
        radio.addEventListener("click", function() {
          if (answer.checked) {
            answer = solution.id;
          }  
        });
      return choicesDiv;
  }
  function create_buttons(question) {

    const questionCheckDiv = document.createElement("div");
    questionCheckDiv.id= question.id + "answer";
    const checkButton = document.createElement('button');
    checkButton.id = question.id + "checkAnswer";
    checkButton.innerHTML = "Check Answer";
    checkButton.setAttribute(
      'style',
      'color: blue; width: 150px; height: 40px; ',
    );
    checkButton.onclick = function () {
      // TODO: Call checkanswer rest API ; if score = 0 (result from the API), display incorrect; else display correct
      // how to get question and answer from user to this function
      const questId = questionIdList[currentPageIndex];
      const answer = 'TODO: needtocheckradiobuttonvalue';
      onCheckAnswer(questId, answer);
    };
    questionCheckDiv.appendChild(checkButton);  // add "yes button" to yes cell

    const continueButton = document.createElement('button');
    continueButton.id = question.id + "continueButton";
    continueButton.innerHTML = "Continue";
    continueButton.setAttribute(
      'style',
      'color: blue; width: 150px; height: 40px;margin-left:10px;',
    );

    continueButton.onclick = function () { 
      const currentPageId = questionIdList[currentPageIndex];
      
      var cur = document.getElementById(currentPageId);
      cur.style.display = 'none';

      const nextIdx = ++currentPageIndex;
      if (nextIdx >= questionIdList.length ) {
        // TODO: enable final page - redirect to endpage using href
        console.log('done with all questions');
        return;
      }
      const nextPageId = questionIdList[nextIdx];
      console.log('next page ', nextPageId);
      var next = document.getElementById(nextPageId);
      next.removeAttribute('hidden');
      next.style.display = 'block';
      
    };
    questionCheckDiv.appendChild(continueButton); 
    return (questionCheckDiv);

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
