import pdfplumber
import requests

# ----------- Extract Text from PDF ------------
def extract_text_from_pdf(pdf_path):
    text = ""
    with pdfplumber.open(pdf_path) as pdf:
        for page in pdf.pages:
            page_text = page.extract_text()
            if page_text:
                text += page_text + "\n"
    return text

# ----------- Ask Ollama (LLaMA2) ----------------
def ask_ollama_llama2(question, context):
    url = "http://localhost:11434/api/chat"
    data = {
        "model": "llama2",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant. Use the given context to answer questions."},
            {"role": "user", "content": f"Context: {context}\n\nQuestion: {question}"}
        ]
    }

    try:
        response = requests.post(url, json=data)
        return response.json()["message"]["content"]
    except Exception as e:
        return f"Error contacting Ollama LLaMA2: {e}"

# ----------- Search Related Links (Web + YouTube) -------------
def search_serpapi(query, api_key):
    url = "https://serpapi.com/search"
    params = {
        "engine": "google",
        "q": query,
        "api_key": api_key,
        "num": 5
    }

    try:
        response = requests.get(url, params=params)
        results = response.json()

        links = []

        if "organic_results" in results:
            for result in results["organic_results"]:
                if "link" in result:
                    links.append(result["link"])

        if "inline_videos" in results:
            for video in results["inline_videos"]:
                if "link" in video:
                    links.append(video["link"])

        return links
    except Exception as e:
        return [f"Error fetching search results: {e}"]

# ----------- Main Interactive Loop ----------------
def main():
    pdf_path = "C:/Users/roshn/Downloads/Cancer_Guide_Understandable.pdf"  # Replace with your PDF path
    serp_api_key = "4c014dacf71ed3132364fbe73311f2e5b1bf14e527a5512e30d9abe62dd00c45"  # Replace with your real key

    print("Extracting PDF content...")
    pdf_text = extract_text_from_pdf(pdf_path)

    while True:
        question = input("\nAsk a question about the PDF (type 'exit' to quit): ")
        if question.lower() == 'exit':
            break

        print("\nAnswering your question using LLaMA2...")
        answer = ask_ollama_llama2(question, pdf_text)
        print("\nAnswer:\n", answer)

        print("\nSearching for related resources and YouTube links...")
        links = search_serpapi(question, serp_api_key)

        if links:
            print("\nRelated Web Resources and YouTube Videos:\n")
            for idx, link in enumerate(links, 1):
                print(f"{idx}. {link}")
        else:
            print("No related links found or SerpAPI error.")

if __name__ == "__main__":
    main()
