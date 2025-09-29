# HR-Portal
```plain text
export APPS_JSON='[
{
"url": "https://github.com/frappe/erpnext",
"branch": "version-14"
},
{
"url": "https://github.com/frappe/hrms.git",
"branch": "version-14"
},
{
"url": "https://gitlab.com/arcapps/frappe_survey.git",
"branch": "master"
}
]'

export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)

docker build \
    --no-cache \
    --build-arg FRAPPE_PATH=https://github.com/frappe/frappe \
    --build-arg FRAPPE_BRANCH=version-14 \
    --build-arg PYTHON_VERSION=3.11.9 \
    --build-arg NODE_VERSION=18.20.2 \
    --build-arg APPS_JSON_BASE64="$APPS_JSON_BASE64" \
    --tag registry.gitlab.com/arcapps/arc-container-registry/hr-portal:1.0.0 \
    --file images/custom/Containerfile \
    . && docker push registry.gitlab.com/arcapps/arc-container-registry/hr-portal:1.0.0

```

```json
{
  "title": "Welcome to Your Career Portal!",
  "description": "",
  "pages": [
    {
      "name": "greetingPage",
      "elements": [
        {
          "type": "html",
          "name": "greetingText1",
          "html": "<h3></h3><p>At <b>Excel Technologies Ltd.</b>, we are committed to helping you take the first step in your career journey. Our platform connects you with a world of opportunities, providing access to job listings from top employers eager to find fresh talent like yours.</p>"
        },
        {
          "type": "html",
          "name": "greetingText2",
          "html": "<h3>Start Your Career Today!</h3><p>Browse a variety of job openings across industries, from internships to full-time positions. Whether you're looking to kick-start your career or gain valuable experience, you’ll find options suited to your skills and aspirations.</p>"
        },
        {
          "type": "html",
          "name": "greetingText3",
          "html": "<h3>Easy Application Process</h3><p>Creating a profile and applying for jobs has never been easier. Upload your resume, complete your profile, and start applying to roles that match your interests and expertise—all with just a few clicks.</p>"
        },
        {
          "type": "radiogroup",
          "name": "developerRole",
          "title": "Select your desired developer role:",
          "choices": [
            "MERN Stack Software Engineer",
            ".NET Core Software Engineer"
          ],
          "isRequired": true
        }
      ]
    },
    {
      "name": "quizPage",
      "elements": [
        {
          "type": "radiogroup",
          "name": "q1",
          "isRequired": true,
          "title": "Which of the following algorithms is most suitable for finding the k smallest elements in an unsorted array, assuming k is much smaller than the size of the array (n)?",
          "choices": [
            "Quick Sort",
            "Heap Sort",
            "Quickselect",
            "Merge Sort"
          ],
          "correctAnswer": "Quickselect"
        },
        {
          "type": "radiogroup",
          "name": "q2",
          "isRequired": true,
          "title": "In a relational database, which normalization form ensures that no table contains non-prime attributes that are transitively dependent on the primary key?",
          "choices": [
            "1NF",
            "2NF",
            "3NF",
            "BCNF"
          ],
          "correctAnswer": "3NF"
        },
        {
          "type": "radiogroup",
          "name": "q3",
          "isRequired": true,
          "title": "What is the result of the following Python code snippet?\n\n```def foo():\n    return [lambda x: x * i for i in range(4)]\n\nfuncs = foo()\nprint([f(2) for f in funcs])```",
          "choices": [
            "[0, 2, 4, 6]",
            "[6, 6, 6, 6]",
            "[8, 8, 8, 8]",
            "[0, 0, 0, 0]"
          ],
          "correctAnswer": "[6, 6, 6, 6]"
        },
        {
          "type": "radiogroup",
          "name": "q4",
          "isRequired": true,
          "title": "Which of the following page replacement algorithms is considered optimal but is not feasible for implementation?",
          "choices": [
            "Least Recently Used (LRU)",
            "First-In-First-Out (FIFO)",
            "Optimal Page Replacement",
            "Clock Replacement"
          ],
          "correctAnswer": "Optimal Page Replacement"
        },
        {
          "type": "radiogroup",
          "name": "q5",
          "isRequired": true,
          "title": "Which transport layer protocol feature ensures reliable delivery of packets in a network?",
          "choices": [
            "Sliding Window Protocol",
            "Three-Way Handshake",
            "Packet Sequencing",
            "All of the above"
          ],
          "correctAnswer": "All of the above"
        },
        {
          "type": "radiogroup",
          "name": "q6",
          "isRequired": true,
          "title": "Which of the following software development models best fits a project with unclear requirements that may evolve over time?",
          "choices": [
            "Waterfall",
            "V-Model",
            "Agile",
            "Spiral"
          ],
          "correctAnswer": "Agile"
        },
        {
          "type": "radiogroup",
          "name": "q7",
          "isRequired": true,
          "title": "What does a SQL injection attack exploit in a web application?",
          "choices": [
            "Unsecured API endpoints",
            "Flaws in input validation",
            "Weak encryption algorithms",
            "Cross-site scripting vulnerabilities"
          ],
          "correctAnswer": "Flaws in input validation"
        },
        {
          "type": "radiogroup",
          "name": "q8",
          "isRequired": true,
          "title": "Which of the following metrics is most appropriate for evaluating a classification model when the dataset is highly imbalanced?",
          "choices": [
            "Accuracy",
            "Precision",
            "F1 Score",
            "Mean Squared Error"
          ],
          "correctAnswer": "F1 Score"
        },
        {
          "type": "radiogroup",
          "name": "q9",
          "isRequired": true,
          "title": "In a distributed system, the CAP theorem states that it is impossible to achieve all three properties simultaneously. Which property refers to the consistency of data across nodes?",
          "choices": [
            "Consistency",
            "Availability",
            "Partition Tolerance",
            "Reliability"
          ],
          "correctAnswer": "Consistency"
        },
        {
          "type": "radiogroup",
          "name": "q10",
          "isRequired": true,
          "title": "What is the time complexity of searching for an element in a balanced binary search tree (e.g., AVL or Red-Black Tree)?",
          "choices": [
            "O(1)",
            "O(log n)",
            "O(n)",
            "O(n log n)"
          ],
          "correctAnswer": "O(log n)"
        }
      ]
    },
    {
      "name": "userInfoPage",
      "elements": [
        {
          "type": "text",
          "name": "userName",
          "title": "Please enter your name:",
          "isRequired": true
        },
        {
          "type": "text",
          "name": "userEmail",
          "title": "Please enter your email:",
          "isRequired": true
        }
      ]
    }
  ]
}

```
