# OCR-read-PDF
Using OCR to read PDF files

# First install requirements:

!sudo apt install tesseract-ocr 4.0.0
!sudo apt-get install tesseract-ocr-eng
!sudo apt-get install tesseract-ocr-ara
!pip install pytesseract
!pip install pdf2image
!apt-get install poppler-utils 
!pip install PIL
!pip install Pillow 4.0.0
!pip install cleantext

import PIL
import os
from pdf2image import convert_from_path
import cleantext
import pytesseract
import os
import cv2

# Code to get the text from file
# Get lecture file from user:
inputFile = input("Enter path of lecture file: ")
pdf_file=inputFile


custom_config=r'--oem 3'
# Convert PDF_pages to Images
pages = convert_from_path(pdf_file, dpi=200,fmt='jpeg')

# String to store the extracted text from images
final_txt=''

# Convert image to grayscale image
def get_grayscale(image):
    return cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

# Remove noise from image
def remove_noise(image):
    return cv2.medianBlur(image,5)
 
# Get thresholding image
def thresholding(image):
    return cv2.threshold(image, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)[1]

# For loop to process each image and do different operations on it
for idx,page in enumerate(pages):
  # First save the image
  img = page.save('img'+str(idx)+'.jpg', 'JPEG')

  # Read saved image
  image=cv2.imread('img'+str(idx)+'.jpg')

# Improve the images quality
  # Get the grayscale image
  gray = get_grayscale(image)
  # Get the blur image
  blur = remove_noise(gray)
  # Get the thresholding image
  thresh = thresholding(blur)
  # Store the final image 
  final_image=thresh
  
  # Using pytesseract to extract the text from stored image
  ocr_result_original= pytesseract.image_to_string(final_image,config=custom_config)
  
  # Save the extracted text
  final_txt=final_txt+ocr_result_original
  
  # Remove the image to keep the memory
  os.remove('img'+str(idx)+'.jpg')
# you can remove the next two lines (only used to see result)
print(final_txt)
print('...............................')

# To clean the extracted text
#clean text will be saved in database to generate the question later
clean_text = cleantext.clean(final_txt,  # text to be cleaned
clean_all= False ,  # Execute all cleaning operations
extra_spaces=True ,  # Remove extra white spaces 
lowercase=True ,  # Convert to lowercase
stp_lang='english'  # Language for stop words
)

#if you do not need to save clean text in text file you can remove the next lines
# Save the clean text to file 'clean_text.txt'
with open('clean_text.txt', 'w') as f:
    f.write(clean_text)
print(clean_text)


