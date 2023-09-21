1. Downloading a blob from Azure Storage using a shared access signature (SAS) URL.
2. Uploading a JSON file to Azure Blob Storage and generating a SAS URL for it.

Before we start, you'd need the Azure SDK for C++ for Blob Storage. If you haven't set it up yet, please follow the official Microsoft documentation to get the SDK and necessary dependencies in place.

Once you're set up:

### 1. Downloading a Blob using a SAS URL:

```cpp
#include <iostream>
#include <azure/storage/blobs.hpp>

void downloadBlobWithSAS(const std::string& sasUrl) {
    Azure::Storage::Blobs::BlobClient blobClient(sasUrl);

    auto response = blobClient.Download();
    Azure::Core::IO::BodyStream* stream = response->Body.get();

    std::vector<uint8_t> blobContent(static_cast<size_t>(response->BlobSize));
    stream->Read(&blobContent[0], blobContent.size());

    std::string content(blobContent.begin(), blobContent.end());
    std::cout << "Blob content: " << content << std::endl;
}

int main() {
    std::string sasUrl = "YOUR_SAS_URL_HERE";
    downloadBlobWithSAS(sasUrl);
    return 0;
}
```

### 2. Uploading a JSON File and Generating a SAS URL:

First, ensure you have your Azure Storage connection string ready:

```cpp
#include <iostream>
#include <azure/storage/blobs.hpp>
#include <fstream>
#include <nlohmann/json.hpp>

void uploadJsonAndGenerateSAS(const std::string& connectionString, const std::string& containerName, const std::string& blobName, const std::string& filePath) {
    // Initialize Blob Container Client
    Azure::Storage::Blobs::BlobContainerClient containerClient(connectionString, containerName);
    Azure::Storage::Blobs::BlobClient blobClient = containerClient.GetBlobClient(blobName);

    // Read the JSON file
    std::ifstream jsonFile(filePath, std::ios::binary);
    blobClient.Upload(jsonFile, Azure::Storage::Blobs::UploadBlobOptions());

    // Generate the SAS token
    Azure::Storage::Sas::BlobSasBuilder sasBuilder;
    sasBuilder.SetPermissions(Azure::Storage::Sas::BlobSasPermissions::Read);
    sasBuilder.SetExpiryTime(Azure::Core::DateTime::Now() + std::chrono::hours(24));
    sasBuilder.SetBlobContainerName(containerClient.GetBlobContainerName());
    sasBuilder.SetBlobName(blobClient.GetBlobName());

    std::string sasToken = blobClient.GenerateSas(sasBuilder);

    std::cout << "SAS URL: " << blobClient.GetUrl() << "?" << sasToken << std::endl;
}

int main() {
    std::string connectionString = "YOUR_AZURE_STORAGE_CONNECTION_STRING";
    std::string containerName = "YOUR_CONTAINER_NAME";
    std::string blobName = "YOUR_BLOB_NAME.json";
    std::string filePath = "PATH_TO_YOUR_JSON_FILE";

    uploadJsonAndGenerateSAS(connectionString, containerName, blobName, filePath);

    return 0;
}
```

Note:
- The `nlohmann/json.hpp` header is for the popular JSON library for C++. If you don't have it, you can obtain it through most package managers or directly from its GitHub repository.
- Replace the placeholders like "YOUR_SAS_URL_HERE", "YOUR_AZURE_STORAGE_CONNECTION_STRING", etc. with your actual data.
- Make sure to handle exceptions, especially for I/O and Azure operations, for a more robust application.