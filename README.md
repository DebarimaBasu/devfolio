import React, { useState } from "react";
import {
  IconChevronRight,
  IconFileUpload,
  IconProgress,
} from "@tabler/icons-react";
import { useLocation, useNavigate } from "react-router-dom";
import { useStateContext } from "../../context/index.jsx";
import ReactMarkdown from "react-markdown";
import FileUploadModal from "./components/file-upload-modal";
import RecordDetailsHeader from "./components/record-details-header";
import { GoogleGenerativeAI } from "@google/generative-ai";

const geminiApiKey = import.meta.env.VITE_GEMINI_API_KEY;

function SingleRecordDetails() {
  const { state } = useLocation();
  const navigate = useNavigate();
  const [file, setFile] = useState(null);
  const [uploading, setUploading] = useState(false);
  const [uploadSuccess, setUploadSuccess] = useState(false);
  const [processing, setIsProcessing] = useState(false);
  // const [analysisResult, setAnalysisResult] = useState(
  //   state.analysisResult || "",
  // );
  const [analysisResult, setAnalysisResult] = useState(
    state?.analysisResult || ""
  );
  
  const [filename, setFilename] = useState("");
  const [filetype, setFileType] = useState("");
  const [isModalOpen, setIsModalOpen] = useState(false);

  const { updateRecord } = useStateContext();

  const handleOpenModal = () => {
    setIsModalOpen(true);
  };

  const handleCloseModal = () => {
    setIsModalOpen(false);
  };

  const handleFileChange = (e) => {
    const file = e.target.files[0];
    console.log("Selected file:", file);
    setFileType(file.type);
    setFilename(file.name);
    setFile(file);

    
  };

  // const readFileAsBase64 = (file) => {
  //   return new Promise((resolve, reject) => {
  //     const reader = new FileReader();
  //     reader.onload = () => resolve(reader.result.split(",")[1]);
  //     reader.onerror = reject;
  //     reader.readAsDataURL(file);
  //   });
  // };
  const readFileAsBase64 = (file) => {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = () => resolve(reader.result.split(',')[1]); // Get only base64 part
      reader.onerror = (error) => reject(error);
    });
  };
  

  // const handleFileUpload = async () => {
  //   setUploading(true);
  //   setUploadSuccess(false);

  //   const genAI = new GoogleGenerativeAI(geminiApiKey);

  //   try {
  //     const base64Data = await readFileAsBase64(file);

  //     const imageParts = [
  //       {
  //         inlineData: {
  //           data: base64Data,
  //           mimeType: filetype,
  //         },
  //       },
  //     ];

  //     const model = genAI.getGenerativeModel({ model: "gemini-1.5-pro" });

  //     const prompt = `You are an expert cancer and any disease diagnosis analyst. Use your knowledge base to answer questions about giving personalized recommended treatments.
  //       give a detailed treatment plan for me, make it more readable, clear and easy to understand make it paragraphs to make it more readable
  //       `;

  //     const result = await model.generateContent([prompt, ...imageParts]);
  //     const response = await result.response;
  //     const text = response.text();
  //     setAnalysisResult(text);
  //     const updatedRecord = await updateRecord({
  //       documentID: state.id,
  //       analysisResult: text,
  //       kanbanRecords: "",
  //     });
  //     setUploadSuccess(true);
  //     setIsModalOpen(false); // Close the modal after a successful upload
  //     setFilename("");
  //     setFile(null);
  //     setFileType("");
  //   } catch (error) {
  //     console.error("Error uploading file:", error);
  //     setUploadSuccess(false);
  //   } finally {
  //     setUploading(false);
  //   }
  // };
  const handleFileUpload = async () => {
    setUploading(true);
    setUploadSuccess(false);
  
    try {
      // Read the PDF as Base64
      const base64Data = await readFileAsBase64(file);
  
      // Update the record (or save somewhere)
      const updatedRecord = await updateRecord({
        documentID: state.id,
        pdfFile: base64Data, // Save the PDF as Base64
        fileName: filename,
        mimeType: filetype,
      });
  
      setUploadSuccess(true);
      setIsModalOpen(false); // Close the modal after upload
      setFilename("");
      setFile(null);
      setFileType("");
  
    } catch (error) {
      console.error("Error uploading PDF file:", error);
      setUploadSuccess(false);
    } finally {
      setUploading(false);
    }
  };
  

  // const processTreatmentPlan = async () => {
  //   setIsProcessing(true);

  //   const genAI = new GoogleGenerativeAI(geminiApiKey);

  //   const model = genAI.getGenerativeModel({ model: "gemini-1.5-pro" });

  //   const prompt = `Your role and goal is to be an that will be using this treatment plan ${analysisResult} to create Columns:
  //               - Todo: Tasks that need to be started
  //               - Doing: Tasks that are in progress
  //               - Done: Tasks that are completed
          
  //               Each task should include a brief description. The tasks should be categorized appropriately based on the stage of the treatment process.
          
  //               Please provide the results in the following  format for easy front-end display no quotating or what so ever just pure the structure below:

  //               {
  //                 "columns": [
  //                   { "id": "todo", "title": "Todo" },
  //                   { "id": "doing", "title": "Work in progress" },
  //                   { "id": "done", "title": "Done" }
  //                 ],
  //                 "tasks": [
  //                   { "id": "1", "columnId": "todo", "content": "Example task 1" },
  //                   { "id": "2", "columnId": "todo", "content": "Example task 2" },
  //                   { "id": "3", "columnId": "doing", "content": "Example task 3" },
  //                   { "id": "4", "columnId": "doing", "content": "Example task 4" },
  //                   { "id": "5", "columnId": "done", "content": "Example task 5" }
  //                 ]
  //               }
                            
  //               `;

  //   const result = await model.generateContent(prompt);
  //   const response = await result.response;
  //   const text = response.text();
  //   const parsedResponse = JSON.parse(text);

  //   console.log(text);
  //   console.log(parsedResponse);
  //   const updatedRecord = await updateRecord({
  //     documentID: state.id,
  //     kanbanRecords: text,
  //   });
  //   console.log(updatedRecord);
  //   navigate("/screening-schedules", { state: parsedResponse });
  //   setIsProcessing(false);
  // };

  return (
    <div className="flex flex-wrap gap-[26px]">
      <button 
        type="button"
        onClick={handleOpenModal}
        className="mt-6 inline-flex items-center gap-x-2 rounded-full border border-green-400 bg-[#17171e] px-4 py-2 text-sm font-medium text-white shadow-sm hover:bg-green-800 disabled:pointer-events-none disabled:opacity-50 dark:border-neutral-700 dark:bg-[#13131a] dark:text-white dark:hover:bg-neutral-800"
      >
        <IconFileUpload />
        Upload Reports
      </button>
      <FileUploadModal
        isOpen={isModalOpen}
        onClose={handleCloseModal}
        onFileChange={handleFileChange}
        onFileUpload={handleFileUpload}
        uploading={uploading}
        uploadSuccess={uploadSuccess}
        filename={filename}
      />
      <RecordDetailsHeader recordName={state.recordName} />
     
          </div>
       
      
    
  );
}

export default SingleRecordDetails;

