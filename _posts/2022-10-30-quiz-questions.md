---
title: Quiz Questions
layout: default
description: Our Frontend talking to Backend Python application serving questions.  This api allows us to get customer responses. 
permalink: /data/quiz
image: /images/feedback.jpeg
tags: [javascript, fetch, dom, getElementID, appendChild]
---
<!-- TODO: add this link to each quiz type href (see continueButton): quiz?subject=APStats&totalQs=5 !--> 
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
  const urllocal = "http://localhost:5000/api/quiz" ;
  
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
  questionIdList = [];
  choiceMap = {};
  selectedAnswer = null;
  myAnswerResponse = {};

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
      answer: selectedAnswer
    };
    const post_options = { 
      ...options, 
      method: 'POST',
      body: JSON.stringify(requestData) 
    }; // clones and replaces method

    post_url = '/checkanswer';
    // fetch the API
    fetch(url + post_url, post_options)
    // response is a RESTful "promise" on any successful fetch
    .then(response => {
      // check for response errors
      if (response.status !== 200) {
          error("PUT API response failure: " + response.status)
          return;  // api failure
      }
      // valid response will have JSON data
      response.json().then(data => {
           
          myAnswerResponse = data;
          var msg1 = document.getElementById(questId + 'sol-1');
          const score = data['scoreForThisAnswer']
          if (score == 0) {
            msg1.innerHTML = 'Correct!';
          }
          else {
            msg1.innerHTML = 'Incorrect!';
          }

          var msg2 = document.getElementById(questId + 'sol-2');
          msg2.innerHTML =  `Your score is ${score}`;

          var msg3 = document.getElementById(questId + 'sol-3');
          msg3.innerHTML =  data['solution'];
          
      })
    })
    // catch fetch errors (ie Nginx ACCESS to server blocked)
    .catch(err => {
      error(err + " " + post_url);
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
      const solution = create_solution(question);
      questionDiv.appendChild(choices);
      questionDiv.appendChild(buttons);
      questionDiv.appendChild(solution);
      
      resultContainer.appendChild(questionDiv);
      
    } // end of questions
  }
  function create_choices(question)  {
      
      // loop through choices to make MCs
      const choicesDiv = document.createElement("div");
      choicesDiv.id= question.id + "choices";
      idx = 0;
      for (const questionChoice of question.choices) {
        idx++;
        var radioDiv = document.createElement("div");
        
        var radioButton = document.createElement("INPUT");
        radioButton.setAttribute("type", "radio");
        radioButton.setAttribute('name', question.id + "choices");
        radioButton.id = question.id + '-radio-'+ idx;

        var labelValue = document.createElement('label');
        labelValue.id = question.id + '-radiolabel-' + idx;
        labelValue.innerHTML = questionChoice;
        labelValue.setAttribute('style', 'margin-left:5px');

        choiceMap[radioButton.id] = questionChoice;
        radioDiv.appendChild(radioButton);
        radioDiv.appendChild(labelValue);
        choicesDiv.appendChild(radioDiv);
        
        radioButton.addEventListener("click", function() {
          if (this.checked) {
            selectedAnswer = choiceMap[this.id];
            document.getElementById(question.id + "continueButton").disabled = false;
          }  
        });
        
      };
      return choicesDiv;
  }
  function create_buttons(question) {

    const questionCheckDiv = document.createElement("div");
    questionCheckDiv.id= question.id + "answer";
    questionCheckDiv.setAttribute(
      'style',
      'margin-top:20px;margin-bottom:20px;',
    );
   
    const checkButton = document.createElement('button');
    checkButton.id = question.id + "checkAnswer";
    checkButton.innerHTML = "Check Answer";
    checkButton.setAttribute(
      'style',
      'color: blue; width: 120px; height: 30px; ',
    );
    checkButton.onclick = function () {
      // TODO: Call checkanswer rest API ; if score = 0 (result from the API), display incorrect; else display correc
      // how to get question and answer from user to this function
      const questId = questionIdList[currentPageIndex];
      if (selectedAnswer === null) {
        alert('Please select one of the answers shown');
        return;
      } 
      onCheckAnswer(questId, selectedAnswer);     
    };
    questionCheckDiv.appendChild(checkButton);  // add "yes button" to yes cell

    const continueButton = document.createElement('button');
    continueButton.id = question.id + "continueButton";
    continueButton.innerHTML = "Continue";
    continueButton.disabled = true;
    continueButton.setAttribute(
      'style',
      'color: blue; width: 120px; height: 30px;margin-left:10px;',
    );

    continueButton.onclick = function () { 
      const currentPageId = questionIdList[currentPageIndex];
      
      var cur = document.getElementById(currentPageId);
      cur.style.display = 'none';     
      const nextIdx = ++currentPageIndex;
      if (nextIdx >= questionIdList.length ) {        
        console.log('done with all questions');
        location.href = "quizfinish";
        return;
      }
      const nextPageId = questionIdList[nextIdx];
      console.log('next page ', nextPageId);
      var next = document.getElementById(nextPageId);
      next.removeAttribute('hidden');
      next.style.display = 'block';
      // Check answer again if it's different when user 
      // clicks check answer
      if (myAnswerResponse['yourAnswer'] != selectedAnswer) {
        onCheckAnswer(currentPageId, selectedAnswer);  
      }
      selectedAnswer = null;
    };
    questionCheckDiv.appendChild(continueButton); 
    return (questionCheckDiv);

  }

  function create_solution(question) {

    var sol = document.createElement('div');
    var title = document.createElement('span');
    title.innerHTML = 'Answer and Solution';
    title.setAttribute('style', 'font-weight:bold; font-size: 15px;');
    sol.appendChild(title);
   
    var msg1 = document.createElement('div');
    msg1.id = question.id + 'sol-1';
    sol.appendChild(msg1);

    var msg2 = document.createElement('div');
    msg2.id = question.id  + 'sol-2';
    sol.appendChild(msg2);

    var msg3 = document.createElement('div');
    msg3.id = question.id  + 'sol-3';
    msg3.setAttribute('style', 'margin-top:5px;');
    sol.appendChild(msg3);

    sol.setAttribute('style', 'padding:10px; border: 1px solid #969696;');

    return sol;
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
