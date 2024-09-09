# Bits de IA - Demo Gemini Nano via Chrome Canary (JavaScript)

## Intro

Esta es una demostración del uso de **Gemini Nano** a través de **Chrome Canary** utilizando JavaScript. Este proyecto muestra cómo integrar funcionalidades de IA en aplicaciones web.

## Links

1. [Instalar Chrome Canary](https://www.google.com/chrome/canary/)
2. [Chrome for Developer : Built-in AI](https://developer.chrome.com/docs/ai/built-in?authuser=1)
3. [Google Built-in AI Early Preview Program](https://docs.google.com/document/d/1WZlAvfrIWDwzQXdqIcCOTcrWLGGgmoesN1VGFbKU_D4/edit?pli=1)
4. [Specs API](https://github.com/WICG/writing-assistance-apis/blob/main/README.md)

## Code demo

```javascript
// Function to create and show the loading spinner
function showSpinner() {
    const spinner = document.createElement('div');
    spinner.id = 'spinner';
    spinner.className = 'spinner';
    document.body.appendChild(spinner);
}

// Function to hide and remove the loading spinner
function hideSpinner() {
    const spinner = document.getElementById('spinner');
    if (spinner) {
        spinner.remove();
    }
}

// Add CSS for spinner and popup
const style = document.createElement('style');
style.textContent = `
    .spinner {
        position: fixed;
        top: 50%;
        left: 50%;
        width: 50px;
        height: 50px;
        border: 5px solid #f3f3f3;
        border-top: 5px solid #3498db;
        border-radius: 50%;
        animation: spin 1s linear infinite;
    }
    @keyframes spin {
        0% { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
    }
    .popup {
        position: fixed;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        background-color: white;
        padding: 20px;
        border-radius: 5px;
        box-shadow: 0 0 10px rgba(0,0,0,0.3);
        z-index: 9999;
        max-width: 80%;
        max-height: 80%;
        overflow-y: auto;
    }
    .popup textarea {
        width: 100%;
        height: 100px;
        margin: 5px 0;
    }
    .popup .selected-text {
        margin-bottom: 10px;
        font-style: italic;
        border-left: 3px solid #3498db;
        padding-left: 10px;
    }
    .popup .close-button {
        position: absolute;
        top: 5px;
        right: 5px;
        cursor: pointer;
        font-size: 20px;
    }
`;
document.head.appendChild(style);

// Function to get the selected text
function getSelectedText() {
    let text = "";
    if (window.getSelection) {
        text = window.getSelection().toString();
    } else if (document.selection && document.selection.type != "Control") {
        text = document.selection.createRange().text;
    }
    return text;
}

// Function to send the selected text to the AI writer API
async function sendToAIWriter(selectedText, prompt) {
    try {
        const writer = await ai.writer.create({
            tone: "formal",
            format: "markdown",
            sharedContext: selectedText,
        });

        const finalResult = await writer.write(prompt);

        return finalResult;
    } catch (error) {
        console.error(error);
        throw error;
    }
}

// Global variable to track if the popup is open
let isPopupOpen = false;

// Function to create and show the popup
function showPopup(selectedText) {
    if (isPopupOpen) return; // Prevent multiple popups

    isPopupOpen = true;
    const popup = document.createElement('div');
    popup.className = 'popup';

    const closeButton = document.createElement('span');
    closeButton.textContent = '×';
    closeButton.className = 'close-button';
    closeButton.onclick = () => {
        popup.remove();
        isPopupOpen = false;
    };

    const selectedTextDiv = document.createElement('div');
    selectedTextDiv.className = 'selected-text';
    selectedTextDiv.textContent = `**** ${selectedText} ****`;

    const promptTextarea = document.createElement('textarea');
    promptTextarea.placeholder = 'Ingrese su prompt';

    const submitButton = document.createElement('button');
    submitButton.textContent = 'Enviar';

    const resultDiv = document.createElement('div');
    resultDiv.className = 'result';

    popup.appendChild(closeButton);
    popup.appendChild(selectedTextDiv);
    popup.appendChild(promptTextarea);
    popup.appendChild(submitButton);
    popup.appendChild(resultDiv);

    document.body.appendChild(popup);

    submitButton.addEventListener('click', async () => {
        const prompt = promptTextarea.value.trim();
        if (prompt) {
            resultDiv.textContent = '';
            const spinnerElement = document.createElement('div');
            spinnerElement.className = 'spinner';
            resultDiv.appendChild(spinnerElement);

            try {
                const result = await sendToAIWriter(selectedText, prompt);
                resultDiv.textContent = result;
            } catch (error) {
                resultDiv.textContent = 'Error: ' + error.message;
            }
        }
    });
}

// Updated event listener for text selection
document.addEventListener('mouseup', (event) => {
    const selectedText = window.getSelection().toString().trim();
    if (selectedText && !isPopupOpen) {
        if (!event.target.closest('.popup')) {
            showPopup(selectedText);
        }
    }
});
