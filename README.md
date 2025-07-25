import React, { useState, useEffect, useRef } from 'react';

// Main App component for the Gemini Chatbot
function App() {
  // State to store the current input message
  const [input, setInput] = useState('');
  // State to manage loading indicator during API calls
  const [isLoading, setIsLoading] = useState(false);
  // State to indicate if Sasha is currently speaking
  const [isSpeaking, setIsSpeaking] = useState(false);
  // State to display status messages to the user
  const [statusMessage, setStatusMessage] = useState("Саша ждет вас..."); // Sasha is waiting for you...

  // Function to send a message to the Gemini API
  const sendMessage = async () => {
    if (input.trim() === '') return; // Не отправляйте пустые сообщения

    const userMessageText = input; // Получите сообщение пользователя
    setInput(''); // Очистите поле ввода
    setIsLoading(true); // Покажите индикатор загрузки
    setStatusMessage("Саша думает..."); // Sasha is thinking...

    try {
      // История чата для API-запроса (только текущее сообщение пользователя, так как текст не отображается)
      let chatHistory = [{ role: "user", parts: [{ text: userMessageText }] }];

      // Сформируйте API-запрос
      const payload = { contents: chatHistory };
      // API-ключ, предоставленный пользователем
      const apiKey = "AIzaSyDLQKwsc9iudSz7mzYWsFe1U6x0Eo35aZs";
      // Конечная точка Gemini API
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

      // Отправьте запрос к Gemini API
      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      const result = await response.json();

      // Проверьте, содержит ли ответ действительный контент
      if (result.candidates && result.candidates.length > 0 &&
          result.candidates[0].content && result.candidates[0].content.parts &&
          result.candidates[0].content.parts.length > 0) {
        const aiText = result.candidates[0].content.parts[0].text;
        speakText(aiText); // Озвучьте ответ ИИ
      } else {
        setStatusMessage("Извините, не удалось получить ответ. Пожалуйста, попробуйте еще раз.");
        console.error("Неожиданная структура ответа API:", result);
      }
    } catch (error) {
      console.error("Произошла ошибка:", error);
      setStatusMessage("Произошла ошибка подключения. Пожалуйста, проверьте ваше интернет-соединение.");
    } finally {
      setIsLoading(false); // Скройте индикатор загрузки
    }
  };

  // Function to speak the given text using Web Speech API
  const speakText = (text) => {
    if ('speechSynthesis' in window) {
      const utterance = new SpeechSynthesisUtterance(text);
      utterance.lang = 'ru-RU'; // Устанавливаем русский язык
      // Fallback to English if Russian is not supported or fails
      utterance.onerror = (event) => {
        console.error('SpeechSynthesisUtterance.onerror', event);
        if (utterance.lang === 'ru-RU') {
            utterance.lang = 'en-US';
            window.speechSynthesis.speak(utterance);
        }
      };

      utterance.onstart = () => {
        setIsSpeaking(true);
        setStatusMessage("Саша говорит..."); // Sasha is speaking...
      };
      utterance.onend = () => {
        setIsSpeaking(false);
        setStatusMessage("Саша ждет вас..."); // Sasha is waiting for you...
      };

      // Cancel any ongoing speech before starting new one
      window.speechSynthesis.cancel();
      window.speechSynthesis.speak(utterance);
    } else {
      setStatusMessage("Извините, ваш браузер не поддерживает синтез речи.");
      console.warn("Web Speech API не поддерживается в этом браузере.");
    }
  };

  // Handle Enter key press in the input field
  const handleKeyPress = (e) => {
    if (e.key === 'Enter' && !isLoading && !isSpeaking) {
      sendMessage();
    }
  };

  return (
    <div
      className="flex flex-col min-h-screen w-full bg-black font-sans antialiased bg-cover bg-center"
      style={{ backgroundImage: `url('https://placehold.co/1920x1080/222222/AAAAAA')` }}
    >
      {/* Chat header */}
      <header className="bg-gradient-to-r from-blue-600 to-purple-700 text-white p-4 shadow-md rounded-b-lg">
        <h1 className="text-2xl font-bold text-center">Саша</h1>
      </header>

      {/* Status display area */}
      <div className="flex-1 flex items-center justify-center p-4">
        <div className="text-white text-lg font-medium text-center p-4 bg-gray-800 bg-opacity-70 rounded-lg shadow-xl">
          {isLoading ? (
            <div className="flex items-center justify-center space-x-2">
              <div className="w-3 h-3 bg-blue-400 rounded-full animate-bounce"></div>
              <div className="w-3 h-3 bg-blue-400 rounded-full animate-bounce delay-75"></div>
              <div className="w-3 h-3 bg-blue-400 rounded-full animate-bounce delay-150"></div>
              <span>{statusMessage}</span>
            </div>
          ) : (
            <span>{statusMessage}</span>
          )}
        </div>
      </div>

      {/* Message input area */}
      <div className="p-4 bg-white border-t border-gray-200 shadow-lg rounded-t-lg flex items-center space-x-3">
        <input
          type="text"
          className="flex-1 p-3 border border-gray-300 rounded-full focus:outline-none focus:ring-2 focus:ring-blue-500 text-gray-700 placeholder-gray-400"
          placeholder="Напишите сообщение..."
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyPress={handleKeyPress}
          disabled={isLoading || isSpeaking} // Disable input when loading or speaking
        />
        <button
          onClick={sendMessage}
          className="bg-blue-600 hover:bg-blue-700 text-white p-3 rounded-full shadow-lg transition duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
          disabled={isLoading || isSpeaking} // Disable button when loading or speaking
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            className="h-6 w-6"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
            strokeWidth="2"
          >
            <path
              strokeLinecap="round"
              strokeLinejoin="round"
              d="M14 5l7 7m0 0l-7 7m7-7H3"
            />
          </svg>
        </button>
      </div>
    </div>
  );
}

export default App;
