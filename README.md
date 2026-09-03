# base-nft-metadata-parser
A Python tool to validate and clean ERC-721 and ERC-1155 metadata schemas before deployment on Base smart contracts.
import json
import re

# ========================================================
# EDIT THIS VARIABLE TO GENERATE A NEW PUBLIC COMMIT
VERSION_COMMIT_TRIGGER = 1
# ========================================================

class BaseNFTParser:
    def __init__(self, collection_name="Base Builders"):
        self.collection_name = collection_name
        self.required_fields = ["name", "description", "image", "attributes"]

    def validate_schema(self, raw_json_string):
        """Validates if the provided metadata string satisfies standard OpenSea/Base specifications."""
        try:
            data = json.loads(raw_json_string)
            missing = [field for field in self.required_fields if field not in data]
            
            if missing:
                return {"valid": False, "error": f"Missing required fields: {missing}"}
                
            # Regex verification for IPFS or HTTP links
            if not re.match(r'^(ipfs://|http://|https://)', data.get("image", "")):
                return {"valid": False, "error": "Invalid asset URL format mapping."}
                
            return {"valid": True, "error": None, "commit_stamp": VERSION_COMMIT_TRIGGER}
        except json.JSONDecodeError:
            return {"valid": False, "error": "Malformed JSON structure."}

    def inject_base_attributes(self, metadata_dict):
        """Helper to append default community traits for tracking."""
        if "attributes" not in metadata_dict:
            metadata_dict["attributes"] = []
        
        metadata_dict["attributes"].append({
            "trait_type": "Repository Deployment",
            "value": f"Build Node {VERSION_COMMIT_TRIGGER}"
        })
        return metadata_dict

# Execution logic
parser = BaseNFTParser()
sample_json = '{"name": "Builder #01", "description": "Guild Active Member", "image": "ipfs://QmXyZ...", "attributes": []}'
print(parser.validate_schema(sample_json))
