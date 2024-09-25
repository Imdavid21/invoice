<script>
  import { LucidePlus, Trash2, ImageUp, RotateCcw } from 'lucide-svelte';
  import { onMount } from 'svelte';

  let appState = {
    company: { name: 'Sisyphus', logo: '' },
    invoice: {
      number: '2020-05-0001',
      created: '03/05/2020',
      due: '18/05/2020',
      from: '789/1 Sector-2c, 38200 Gandhinagar, France',
      fromTaxIdName: 'SIRET',
      fromTaxId: '362 521 879 00034',
      fromContact: { mail: 'contact@betao.se', phone: '848712194' },
      to: 'Willy Wonka, 1445 West Norwood Avenue, Itasca, Illinois, USA',
      toTaxIdName: 'SIRET',
      toTaxId: '362 521 879 00034',
      toContact: { mail: 'om@om.com', phone: '97223041054' }
    },
    items: [
      { desc: 'Nom du produit', price: 20, quantity: 150 },
      { desc: 'Nom du produit', price: 20, quantity: 150 },
      { desc: 'Nom du produit', price: 20, quantity: 150 }
    ],
    taxPercent: 0,
    discountPercent: 0,
    note: 'This is a custom message that might be relevant to the customer. It can span up to three or four rows.',
    payment: { accountNumber: '9876 5432 1098', accountName: 'Sisyphus', bank: 'Bank Name' },
    currency: 'EUR'
  };

  function getCurrencySymbol(code) {
    return code === 'EUR' ? '€' : '';
  }

  function save() {
    localStorage.setItem('invoiceData', JSON.stringify(appState));
  }

  function addItem() {
    appState.items = [...appState.items, { desc: 'New Item', price: 0, quantity: 1 }];
    save();
  }

  function deleteItem(index) {
    appState.items = appState.items.filter((_, i) => i !== index);
    save();
  }

  $: subTotal = appState.items.reduce((total, item) => total + item.price * item.quantity, 0);
  $: totalDue = subTotal - (subTotal * (+appState.discountPercent || 0) / 100) + (subTotal * (+appState.taxPercent || 0) / 100);

  onMount(() => {
    const savedData = localStorage.getItem('invoiceData');
    if (savedData) {
      appState = JSON.parse(savedData);
    }
  });
</script>

<style>
  .invoice-container {
    max-width: 800px;
    margin: auto;
    background: #fff;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    font-family: Arial, sans-serif;
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 2px solid #eaeaea;
    padding-bottom: 20px;
  }

  .invoice-info {
    display: flex;
    justify-content: space-between;
    padding: 20px 0;
    border-bottom: 2px solid #eaeaea;
  }

  .details-grid {
    display: grid;
    grid-template-columns: 1fr 2fr;
    gap: 20px;
  }

  .table {
    width: 100%;
    margin-top: 20px;
    border-collapse: collapse;
  }

  .table th, .table td {
    padding: 12px;
    border: 1px solid #ddd;
    text-align: left;
  }

  .total-section {
    margin-top: 20px;
    display: flex;
    justify-content: flex-end;
  }

  .total-table {
    width: 300px;
  }

  .footer {
    text-align: center;
    font-size: 12px;
    margin-top: 20px;
    color: #777;
  }
</style>

<div class="invoice-container">
  <!-- Header Section -->
  <div class="header">
    <div>
      <img src={appState.company.logo} alt="Company Logo" class="max-h-18 max-w-40 object-cover m-0" />
      <h2 class="font-bold">{appState.company.name}</h2>
      <p>John Brandon</p>
      <p>{appState.invoice.from}</p>
      <p>{appState.invoice.fromContact?.mail} | {appState.invoice.fromContact?.phone}</p>
      <p>SIRET: {appState.invoice.fromTaxId}</p>
      <p>TVA: {appState.invoice.fromTaxId}</p>
    </div>
    <div>
      <h3 class="text-right">#{appState.invoice.number}</h3>
      <h1 class="text-right text-3xl font-bold">{getCurrencySymbol(appState.currency)} {totalDue.toFixed(2)}</h1>
    </div>
  </div>

  <!-- Invoice and Recipient Details Section -->
  <div class="invoice-info">
    <div class="details-grid">
      <div>
        <p>Date de facture: {appState.invoice.created}</p>
        <p>Date de livraison: {appState.invoice.due}</p>
        <p>Conditions de règlement: Sous 15 jours</p>
        <p>Échéance de paiement: {appState.invoice.due}</p>
      </div>
      <div>
        <p><strong>{appState.invoice.to}</strong></p>
        <p>{appState.invoice.toContact?.mail} | {appState.invoice.toContact?.phone}</p>
        <p>SIRET: {appState.invoice.toTaxIdName}</p>
        <p>TVA: {appState.invoice.toTaxId}</p>
        <p>Facture: {appState.note}</p>
      </div>
    </div>
  </div>

  <!-- Item Table -->
  <table class="table">
    <thead>
      <tr>
        <th>N°</th>
        <th>Article</th>
        <th>Quantité</th>
        <th>Prix Unité HT</th>
        <th>TVA</th>
        <th>Montant HT</th>
        <th>Montant TTC</th>
      </tr>
    </thead>
    <tbody>
      {#each appState.items as item, index}
        <tr>
          <td>{index + 1}</td>
          <td>{item.desc}</td>
          <td>{item.quantity}</td>
          <td>{getCurrencySymbol(appState.currency)}{item.price.toFixed(2)}</td>
          <td>0%</td>
          <td>{getCurrencySymbol(appState.currency)}{(item.price * item.quantity).toFixed(2)}</td>
          <td>{getCurrencySymbol(appState.currency)}{(item.price * item.quantity).toFixed(2)}</td>
        </tr>
      {/each}
    </tbody>
  </table>

  <!-- Total Section -->
  <div class="total-section">
    <table class="total-table">
      <tr>
        <td>Total HT</td>
        <td>{getCurrencySymbol(appState.currency)} {subTotal.toFixed(2)}</td>
      </tr>
      <tr>
        <td>Total débours</td>
        <td>{getCurrencySymbol(appState.currency)} {(subTotal * (+appState.discountPercent / 100)).toFixed(2)}</td>
      </tr>
      <tr>
        <td>Total TVA</td>
        <td>{getCurrencySymbol(appState.currency)} {(subTotal * (+appState.taxPercent / 100)).toFixed(2)}</td>
      </tr>
      <tr class="font-bold">
        <td>Total TTC</td>
        <td>{getCurrencySymbol(appState.currency)} {totalDue.toFixed(2)}</td>
      </tr>
    </table>
  </div>

  <!-- Footer Section -->
  <div class="footer">
    <p>termes et conditions</p>
    <p>Please pay within 15 days of receiving this invoice.</p>
  </div>
</div>
