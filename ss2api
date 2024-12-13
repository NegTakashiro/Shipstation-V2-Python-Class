from typing import Dict, List, Optional, Union, Any
from dataclasses import dataclass
from datetime import datetime
import requests
from enum import Enum

# Enums
class BatchStatus(str, Enum):
    CREATING = "creating"
    CREATED = "created"
    ERROR = "error"
    PROCESSING = "processing"
    COMPLETED = "completed"

class SortDir(str, Enum):
    ASC = "asc"
    DESC = "desc"

class LabelDownloadType(str, Enum):
    URL = "url"
    PDF = "pdf"
    ZPL = "zpl"

# Data Models
@dataclass
class Carrier:
    carrier_id: str
    name: str
    code: str
    account_number: Optional[str] = None
    requires_funded_amount: bool = False
    balance: Optional[float] = None
    nickname: Optional[str] = None
    friendly_name: Optional[str] = None
    primary: bool = False

@dataclass
class BatchError:
    error_id: str
    message: str
    details: Optional[Dict] = None

@dataclass
class Batch:
    batch_id: str
    status: BatchStatus
    created_at: datetime
    shipment_count: int
    label_count: int
    error_count: int
    batch_number: Optional[str] = None
    external_batch_id: Optional[str] = None
    batch_notes: Optional[str] = None

class ShipStationAPI:
    def __init__(self, api_key: str, api_secret: str, base_url: str = "https://ssapi.shipstation.com"):
        self.base_url = base_url.rstrip('/')
        self.auth = (api_key, api_secret)
        self.session = requests.Session()
        self.session.auth = self.auth
        self.session.headers.update({
            "Content-Type": "application/json"
        })

    def _make_request(
        self, 
        method: str, 
        endpoint: str, 
        params: Optional[Dict] = None, 
        json: Optional[Dict] = None
    ) -> Any:
        """Make a request to the ShipStation API."""
        url = f"{self.base_url}{endpoint}"
        response = self.session.request(method, url, params=params, json=json)
        response.raise_for_status()
        return response.json() if response.content else None

    # Batch Endpoints
    def list_batches(
        self,
        status: Optional[BatchStatus] = None,
        page: int = 1,
        page_size: int = 25,
        sort_dir: SortDir = SortDir.DESC,
        batch_number: Optional[str] = None,
        created_at_start: Optional[datetime] = None,
        created_at_end: Optional[datetime] = None
    ) -> List[Batch]:
        """List all batches with optional filters."""
        params = {
            "page": page,
            "pageSize": page_size,
            "sortDir": sort_dir.value,
        }
        if status:
            params["status"] = status.value
        if batch_number:
            params["batchNumber"] = batch_number
        if created_at_start:
            params["createdAtStart"] = created_at_start.isoformat()
        if created_at_end:
            params["createdAtEnd"] = created_at_end.isoformat()
            
        return self._make_request("GET", "/v2/batches", params=params)

    def get_batch_by_external_id(self, external_batch_id: str) -> Batch:
        """Get a batch by its external ID."""
        return self._make_request("GET", f"/v2/batches/external_batch_id/{external_batch_id}")

    def get_batch(self, batch_id: str) -> Batch:
        """Get a batch by its ID."""
        return self._make_request("GET", f"/v2/batches/{batch_id}")

    def create_batch(
        self,
        shipment_ids: Optional[List[str]] = None,
        rate_ids: Optional[List[str]] = None,
        external_batch_id: Optional[str] = None,
        batch_notes: Optional[str] = None
    ) -> Batch:
        """Create a new batch."""
        request_body = {
            "shipmentIds": shipment_ids or [],
            "rateIds": rate_ids or []
        }
        if external_batch_id:
            request_body["externalBatchId"] = external_batch_id
        if batch_notes:
            request_body["batchNotes"] = batch_notes
            
        return self._make_request("POST", "/v2/batches", json=request_body)

    def delete_batch(self, batch_id: str) -> None:
        """Delete a batch."""
        self._make_request("DELETE", f"/v2/batches/{batch_id}")

    def add_to_batch(self, batch_id: str, shipment_ids: List[str], rate_ids: Optional[List[str]] = None) -> None:
        """Add shipments or rates to a batch."""
        request_body = {
            "shipmentIds": shipment_ids,
            "rateIds": rate_ids or []
        }
        self._make_request("POST", f"/v2/batches/{batch_id}/add", json=request_body)

    def remove_from_batch(self, batch_id: str, shipment_ids: List[str], rate_ids: Optional[List[str]] = None) -> None:
        """Remove shipments or rates from a batch."""
        request_body = {
            "shipmentIds": shipment_ids,
            "rateIds": rate_ids or []
        }
        self._make_request("POST", f"/v2/batches/{batch_id}/remove", json=request_body)

    def list_batch_errors(self, batch_id: str, page: int = 1, page_size: int = 25) -> List[BatchError]:
        """List errors for a batch."""
        params = {
            "page": page,
            "pageSize": page_size
        }
        return self._make_request("GET", f"/v2/batches/{batch_id}/errors", params=params)

    def process_batch(self, batch_id: str, process_request: Dict) -> None:
        """Process a batch for label generation."""
        self._make_request("POST", f"/v2/batches/{batch_id}/process/labels", json=process_request)

    # Carrier Endpoints
    def list_carriers(self) -> List[Carrier]:
        """List all carriers."""
        return self._make_request("GET", "/v2/carriers")

    def get_carrier(self, carrier_id: str) -> Carrier:
        """Get a specific carrier by ID."""
        return self._make_request("GET", f"/v2/carriers/{carrier_id}")

    # Rate Endpoints
    def estimate_rates(self, request_body: Dict) -> Dict:
        """Estimate shipping rates."""
        return self._make_request("POST", "/v2/rates/estimate", json=request_body)

    def calculate_rates(self, request_body: Dict) -> Dict:
        """Calculate shipping rates for a shipment."""
        return self._make_request("POST", "/v2/rates/calculate", json=request_body)

    # Label Endpoints
    def create_label(self, request_body: Dict) -> Dict:
        """Create a shipping label."""
        return self._make_request("POST", "/v2/labels", json=request_body)

    def create_label_from_rate(self, rate_id: str, request_body: Dict) -> Dict:
        """Create a label from a rate."""
        return self._make_request("POST", f"/v2/rates/{rate_id}/label", json=request_body)

    def get_label(self, label_id: str, label_download_type: LabelDownloadType = LabelDownloadType.PDF) -> Dict:
        """Get a label by ID."""
        params = {"labelDownloadType": label_download_type.value}
        return self._make_request("GET", f"/v2/labels/{label_id}", params=params)

    # Manifest Endpoints
    def create_manifest(self, request_body: Dict) -> Dict:
        """Create a manifest."""
        return self._make_request("POST", "/v2/manifests", json=request_body)

    def get_manifest(self, manifest_id: str) -> Dict:
        """Get manifest details."""
        return self._make_request("GET", f"/v2/manifests/{manifest_id}")

    # Pickup Endpoints
    def schedule_pickup(self, request_body: Dict) -> Dict:
        """Schedule a pickup."""
        return self._make_request("POST", "/v2/pickups", json=request_body)

    def get_pickup(self, pickup_id: str) -> Dict:
        """Get pickup details."""
        return self._make_request("GET", f"/v2/pickups/{pickup_id}")

# Example usage:
if __name__ == "__main__":
    api = ShipStationAPI(
        api_key="your_api_key",
        api_secret="your_api_secret"
    )

    try:
        # List carriers
        carriers = api.list_carriers()
        print("Available carriers:", carriers)

        # Create a batch
        batch = api.create_batch(
            shipment_ids=["se-123456", "se-789012"],
            batch_notes="Test batch"
        )
        print("Created batch:", batch)

        # Get batch errors
        errors = api.list_batch_errors(batch.batch_id)
        print("Batch errors:", errors)

    except requests.exceptions.RequestException as e:
        print(f"API Error: {e}") 
